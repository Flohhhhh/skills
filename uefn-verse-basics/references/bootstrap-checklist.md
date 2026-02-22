# Bootstrap Checklist: UEFN + Verse

Use this as the default execution checklist for onboarding requests.

## 1. Environment Readiness

1. Confirm user can install and run Epic Games Launcher.
2. Confirm Fortnite and UEFN are installed and launch successfully.
3. If needed, direct user to Epic prerequisites/install page in `docs-map.md`.

## 2. Create A Starter Island

1. Create a new project from a suitable template.
2. Open the island in UEFN and verify edit access.
3. Save project before adding Verse code.

## 3. Create The First Verse Device

1. Open Verse Explorer.
2. Create a new Verse file/device from template.
3. Paste a minimal `creative_device` class (see `SKILL.md` snippets).
4. Build Verse code and resolve compile errors before continuing.

## 4. Attach Device And Run

1. Place the Verse device in the level if not already placed.
2. Assign `@editable` references in Details panel when used.
3. Launch Session and confirm expected runtime behavior.

## 5. Fast Iteration Loop

1. For Verse-only edits, build then use Push Verse Changes.
2. For broader island/device state changes, use Push Changes.
3. Re-test in session after each incremental change.

## 6. Troubleshooting Order

1. Check Message Log and Verse build diagnostics first.
2. Confirm file compiles with no unresolved symbols/imports.
3. Confirm device is placed and configured in-world.
4. Confirm events/functions are actually wired to gameplay actions.
5. Restart session when state appears stale.

## 7. Team Workflow Baseline

1. Sync latest project state before edits.
2. Check out assets before editing.
3. Check in small, coherent changes frequently.
4. Follow Unreal Revision Control guidance from `docs-map.md`.

