---
name: unreal-verse
description: Guidance for writing, reviewing, and debugging Unreal Editor for Fortnite (UEFN) Verse code using Epic's official language documentation. Use when implementing Verse devices, classes, functions, control flow, containers, failure contexts, specifiers/attributes, modules/imports, effects, concurrency, or translating logic into valid Verse.
---

# Unreal Verse

## Overview

Use Epic documentation as the source of truth for Verse syntax and behavior.
Route each task to the right doc first, then implement with minimal, valid patterns.

## Workflow

1. Classify the task.
2. Open the matching section in `references/verse-doc-index.md`.
3. Draft the smallest valid Verse solution first.
4. Verify effects (`<suspends>`, `<transacts>`, `<decides>`), failure contexts, and container access.
5. Return code plus a short rationale tied to the referenced docs.

## Minimal Common Patterns

Use these as copy-start snippets, then adapt names and imports to the target project.

### Class With Mutable State

```verse
counter := class:
    var total:int = 0

    Add(value:int):void =
        set total += value
```

### Failable Lookup (Option/Failure Context)

```verse
if (first := numbers[0]):
    Print("{first}")
```

### Map Insert + Read

```verse
scores:[string]int = map{}
set scores["alice"] = 10

if (score := scores["alice"]):
    Print("{score}")
```

### Basic Loop

```verse
for (n := 0..2):
    Print("{n}")
```

### Suspending Function

```verse
DoLater()<suspends>:void =
    Sleep(1.0)
```

### Race Two Tasks

```verse
winner := race:
    TaskA()
    TaskB()
```

## Execution Rules

- Prefer explicit parameter and return types.
- Keep functions small and compose behavior from small helpers.
- Treat indexing and map reads as failable operations.
- Use specifiers/attributes only when needed and verify exact syntax in docs.
- Use `race`, `sync`, and `branch` deliberately; always reason about cancellation and lifetime.
- Check language-version notes before using newer syntax in older projects.

## Output Expectations

- Produce code that compiles conceptually against current Verse docs.
- Cite the specific Epic page used for non-trivial constructs.
- Keep examples minimal unless the user asks for full architecture.

