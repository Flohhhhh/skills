---
name: uefn-verse-basics
description: Beginner-focused onboarding and developer workflow for using Verse in Unreal Editor for Fortnite (UEFN). Use when setting up UEFN/Fortnite prerequisites, creating a first Verse-enabled project, creating Verse devices from templates, compiling and playtesting with Launch Session and Push Verse Changes, troubleshooting common setup/build/runtime issues, and establishing basic project/revision-control habits.
---

# UEFN Verse Basics

## Overview

Use this skill for practical "get started and keep moving" support in UEFN + Verse.
Treat Epic docs as source of truth and guide users through a concrete, testable loop.

## Read Order

1. `references/docs-map.md` for source routing.
2. `references/bootstrap-checklist.md` for end-to-end onboarding and DX loop.

## Execution Model

1. Confirm the user stage:
   `install`, `new project`, `first Verse file`, `compile`, `playtest`, `debug`, or `team workflow`.
2. Give only the next actionable step set (avoid dumping all docs at once).
3. Prefer the smallest working loop:
   create Verse file -> build Verse code -> place device -> launch session -> verify behavior -> iterate.
4. If blocked, diagnose by failure class:
   `environment`, `compile`, `session/push`, `device wiring`, `runtime logic`.
5. Always include the exact Epic page link used.

## Minimal UEFN Verse Patterns

Keep these snippets tiny and adapt imports/devices to the target island.

### First Device Skeleton

```verse
using { /Fortnite.com/Devices }
using { /Verse.org/Simulation }
using { /UnrealEngine.com/Temporary/Diagnostics }

hello_world_device := class(creative_device):
    OnBegin<override>()<suspends>:void =
        Print("Hello from UEFN Verse")
```

### Editable Device Reference

```verse
using { /Fortnite.com/Devices }

example_device := class(creative_device):
    @editable
    Button:button_device = button_device{}
```

## DX Guardrails

- State clearly which steps require UEFN GUI interactions.
- In compile loops, prefer `Build Verse Code` first, then `Push Verse Changes` for Verse-only edits.
- After changing placed device behavior, use `Push Changes` when needed so level state/device updates are applied.
- In troubleshooting, start with Message Log + Verse Build status, then validate device placement and editable assignments.
- For collaborative projects, prefer Unreal Revision Control workflows early (sync/check-out/check-in discipline).

