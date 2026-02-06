---
name: polar-sh-nextjs-convex
description: Minimal Polar.sh billing integration for Next.js App Router with Convex as the entitlement authority. Use when implementing subscriptions, one-time purchases, checkout, customer portal, webhooks, and billing-to-entitlement sync in a webhook-driven architecture.
---

# Polar.sh Integration (Next.js + Convex)

Minimal, reusable patterns to connect Polar billing flows to Convex-backed entitlements.

---

## When To Use

Use this skill when your app:

- Needs **subscriptions, one-time purchases, or billing**
- Uses **Next.js App Router**
- Uses **Convex** for backend state and access control
- Wants Polar to handle checkout, portal, payment processing, and lifecycle events

Prefer this pattern when you want billing offloaded to Polar and entitlements centralized in Convex.

---

## What This Integration Provides

- Checkout route
- Customer portal route
- Webhook ingestion route
- Convex mutations for billing sync
- A consistent entitlement source for app authorization

---

## Installation

```bash
pnpm add @polar-sh/sdk @polar-sh/nextjs
```

---

## Environment Variables

**Required:**

```bash
POLAR_ACCESS_TOKEN=
POLAR_WEBHOOK_SECRET=
POLAR_SERVER=sandbox # or production
```

**Optional:**

```bash
POLAR_SUCCESS_URL=
APP_URL=
```

---

## Suggested File Layout

```
src/
  app/
    checkout/route.ts
    portal/route.ts
    api/webhook/polar/route.ts
  lib/polar/client.ts

convex/
  polar.ts
  entitlements.ts
```

---

## Polar Client (Optional Helper)

Use when you need to call Polar APIs directly.

**Location:** `src/lib/polar/client.ts`

```ts
import { Polar } from "@polar-sh/sdk";

export const polar = new Polar({
  accessToken: process.env.POLAR_ACCESS_TOKEN!,
  server: process.env.POLAR_SERVER === "sandbox" ? "sandbox" : "production",
});
```

---

## Checkout Route

Provides hosted checkout from Polar.

**Location:** `src/app/checkout/route.ts`

```ts
import { Checkout } from "@polar-sh/nextjs";

export const GET = Checkout({
  accessToken: process.env.POLAR_ACCESS_TOKEN!,
  successUrl: process.env.POLAR_SUCCESS_URL!,
});
```

**Usage:**
- Redirect users to `/checkout` with query params for product ID and metadata
- Example: `/checkout?productId=prod_xxx&customerId=cust_yyy`

---

## Customer Portal Route

Allows users to manage subscriptions and billing.

**Location:** `src/app/portal/route.ts`

```ts
import { CustomerPortal } from "@polar-sh/nextjs";

export const GET = CustomerPortal({
  accessToken: process.env.POLAR_ACCESS_TOKEN!,
});
```

**Important:** Resolve the current app user to the correct Polar customer before sending users to the portal.

---

## Webhook Route

Handles subscription and billing lifecycle events.

**Location:** `src/app/api/webhook/polar/route.ts`

```ts
import { Webhooks } from "@polar-sh/nextjs";
import { api } from "@/convex/_generated/api";

export const POST = Webhooks({
  webhookSecret: process.env.POLAR_WEBHOOK_SECRET!,

  onCustomerCreated: async (customer) => {
    await fetch(process.env.CONVEX_SITE_URL + "/api/actions", {
      method: "POST",
      body: JSON.stringify({
        path: "polar:syncCustomer",
        args: { customer },
      }),
    });
  },

  onSubscriptionCreated: async (subscription) => {
    await fetch(process.env.CONVEX_SITE_URL + "/api/actions", {
      method: "POST",
      body: JSON.stringify({
        path: "polar:syncSubscription",
        args: { subscription },
      }),
    });
  },

  onSubscriptionUpdated: async (subscription) => {
    // Handle updates (cancellations, plan changes, etc.)
  },

  onOrderCreated: async (order) => {
    // Handle one-time purchases
  },
});
```

**Webhook rules:**

- Keep handlers **idempotent** (safe to process multiple times)
- Call **internal Convex mutations** to update state
- Do **not depend on client auth/session state**
- Log unknown events and payload shapes for debugging

---

## Convex Responsibilities

Convex is the source of truth for:

- **Mirrored billing state** (subscriptions, orders, customers)
- **Entitlements** (access permissions derived from billing)
- **Authorization checks** (frontend and backend)

---

## Convex Mutation Pattern

**Location:** `convex/polar.ts`

```ts
import { internalMutation } from "./_generated/server";
import { v } from "convex/values";

export const syncSubscription = internalMutation({
  args: { subscription: v.any() },
  handler: async (ctx, args) => {
    const { subscription } = args;
    
    // Find or create user by customer ID
    const userId = await ctx.db
      .query("users")
      .withIndex("by_polar_customer_id", (q) =>
        q.eq("polarCustomerId", subscription.customerId)
      )
      .unique();

    if (!userId) {
      throw new Error(`User not found for customer ${subscription.customerId}`);
    }

    // Upsert subscription
    const existing = await ctx.db
      .query("subscriptions")
      .withIndex("by_polar_id", (q) => q.eq("polarId", subscription.id))
      .unique();

    if (existing) {
      await ctx.db.patch(existing._id, {
        status: subscription.status,
        currentPeriodEnd: subscription.currentPeriodEnd,
        productId: subscription.productId,
        updatedAt: Date.now(),
      });
    } else {
      await ctx.db.insert("subscriptions", {
        userId: userId._id,
        polarId: subscription.id,
        status: subscription.status,
        productId: subscription.productId,
        currentPeriodEnd: subscription.currentPeriodEnd,
        createdAt: Date.now(),
        updatedAt: Date.now(),
      });
    }
  },
});

export const syncCustomer = internalMutation({
  args: { customer: v.any() },
  handler: async (ctx, args) => {
    // Link Polar customer to app user
  },
});
```

---

## Entitlement Strategy

Choose one model:

### Option A: Derived Entitlements

Compute access from subscription data at read time.

```ts
// convex/entitlements.ts
import { query } from "./_generated/server";
import { v } from "convex/values";

export const getUserEntitlements = query({
  args: { userId: v.id("users") },
  handler: async (ctx, args) => {
    const activeSubscriptions = await ctx.db
      .query("subscriptions")
      .withIndex("by_user_id", (q) => q.eq("userId", args.userId))
      .filter((q) => q.eq(q.field("status"), "active"))
      .collect();

    return {
      hasPro: activeSubscriptions.some((s) => s.productId === "pro_plan"),
      hasTeam: activeSubscriptions.some((s) => s.productId === "team_plan"),
    };
  },
});
```

**Pros:** Always correct, no duplication
**Cons:** Slightly slower

### Option B: Stored Entitlements

Maintain an `entitlements` table updated by webhook mutations.

```ts
// Update entitlements when subscription changes
await ctx.db.insert("entitlements", {
  userId,
  feature: "pro_access",
  expiresAt: subscription.currentPeriodEnd,
});
```

**Pros:** Faster access checks, simple frontend logic
**Cons:** Requires synchronization logic

---

## Identity Linking

Use stable user IDs, not email-only matching.

**Common patterns:**

1. **Store app user ID in Polar customer metadata**
   - When creating Polar customer, pass `{ metadata: { appUserId: "user_123" } }`
   - In webhooks, extract `customer.metadata.appUserId`

2. **Store Polar customer ID in Convex**
   - Add `polarCustomerId` field to users table
   - Index by `polarCustomerId` for fast lookups

---

## Frontend Access Pattern

1. Query Convex for current entitlements
2. Gate UI with entitlement flags
3. Re-check entitlements in server routes/actions for sensitive operations

**Example:**

```tsx
import { useQuery } from "convex/react";
import { api } from "@/convex/_generated/api";

export function ProFeature() {
  const entitlements = useQuery(api.entitlements.getUserEntitlements);

  if (!entitlements?.hasPro) {
    return <UpgradePrompt />;
  }

  return <ProContent />;
}
```

---

## Operational Checklist

- [ ] Configure webhook endpoint in Polar dashboard
- [ ] Store `POLAR_WEBHOOK_SECRET` securely in environment
- [ ] Subscribe only to required events (avoid unnecessary traffic)
- [ ] Make webhook handlers idempotent
- [ ] Log unknown events and payload shapes for debugging
- [ ] Set up monitoring for webhook failures
- [ ] Test with Polar sandbox environment first

---

## Common Pitfalls

**❌ Assuming one subscription per user**
- Users may have multiple subscriptions (personal + team)
- Always query all active subscriptions

**❌ Running heavy business logic inside webhook handlers**
- Keep webhooks fast and focused on data sync
- Defer complex operations to background jobs

**❌ Skipping environment consistency checks**
- Verify `POLAR_SERVER` matches your environment
- Don't mix sandbox and production data

**❌ Using client-session assumptions in webhooks**
- Webhooks come from Polar servers, not authenticated users
- Use customer/subscription IDs, not session tokens

**❌ Not handling webhook retries**
- Polar retries failed webhooks
- Make handlers idempotent to avoid duplicate processing

---

## Extension Opportunities

This integration can later support:

- **Multiple subscription tiers** (free, pro, enterprise)
- **Usage-based billing** (metered API calls, storage)
- **License / seat tracking** (team subscriptions)
- **Promotional pricing** (coupons, trials)
- **Billing analytics dashboards**
- **Subscription pause/resume flows**

---

## Summary

**Polar** manages billing and payment flows.

**Next.js** provides routing and integration surfaces.

**Convex** manages application state and authorization.

Keep payment flows in Polar. Keep app authorization in Convex. Keep integration logic thin and webhook-driven.
