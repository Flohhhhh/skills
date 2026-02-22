# Verse Docs Index (Epic Official)

Use these pages in this order: quick syntax, full semantics, then topic detail.

## Core Language

- Quick reference (syntax-first):  
  https://dev.epicgames.com/documentation/en-us/fortnite/verse-language-quick-reference
- Full language reference (semantics):  
  https://dev.epicgames.com/documentation/en-us/fortnite/verse-language-reference
- Verse API reference (types/functions in UEFN):  
  https://dev.epicgames.com/documentation/en-us/fortnite/verse-api

## Select By Task

- New to Verse or checking syntax quickly: quick reference.
- Expressions and evaluation behavior:  
  https://dev.epicgames.com/documentation/en-us/fortnite/expressions-in-verse
- Operators and precedence:  
  https://dev.epicgames.com/documentation/en-us/fortnite/operators-in-verse
- Functions and calling patterns:  
  https://dev.epicgames.com/documentation/en-us/fortnite/functions-in-verse
- Control flow (`if`, `for`, branching):  
  https://dev.epicgames.com/documentation/en-us/fortnite/control-flow-in-verse
- Containers (`array`, `map`, ranges, tuples):  
  https://dev.epicgames.com/documentation/en-us/fortnite/container-types-in-verse
- Failure semantics and failable expressions:  
  https://dev.epicgames.com/documentation/en-us/fortnite/failure-in-verse
- Time flow and concurrency (`suspends`, `race`, `sync`, `branch`):  
  https://dev.epicgames.com/documentation/en-us/fortnite/time-flow-and-concurrency-in-verse
- Modules and import paths:  
  https://dev.epicgames.com/documentation/en-us/fortnite/modules-and-paths-in-verse
- Specifiers and attributes:  
  https://dev.epicgames.com/documentation/en-us/fortnite/specifiers-and-attributes-in-verse

## Quality Guardrails

- Verse code style guide:  
  https://dev.epicgames.com/documentation/en-us/fortnite/verse-code-style-guide-in-unreal-editor-for-fortnite
- Language version updates and compatibility notes:  
  https://dev.epicgames.com/documentation/en-us/fortnite/verse-language-version-updates-and-deprecations

## Practical Checks Before Finalizing Code

1. Verify every import path exists in the target project.
2. Verify every failable operation is handled in a failure context.
3. Verify effect specifiers match behavior (`<suspends>`, `<transacts>`, `<decides>`).
4. Verify concurrency blocks have intentional completion/cancellation behavior.
5. Verify style guide naming and formatting for maintainability.

