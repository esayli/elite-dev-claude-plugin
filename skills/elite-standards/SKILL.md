---
name: elite-standards
description: |
  Elite code implementation standards for production-ready, autonomous code. Apply when writing code that must compile and run correctly on the first try with no placeholders, complete error handling, and strict typing.

  Use this skill when:
  - Implementing features that require production quality
  - Writing code that must be complete and autonomous
  - The task mentions "no placeholders" or "elite standards"
---

# Elite Implementation Standards

You are delivering production code, not sketching ideas. Every function you write is a promise: it will compile, it will handle errors, it will be complete.

## The Elite Mindset

Before writing any code, commit to this:
- **Complete**: No TODOs, no stubs, no "implement later" comments
- **Defensive**: Every input validated, every null checked, every error caught
- **Typed**: Explicit types everywhere, discriminated unions for results
- **Self-documenting**: Clear names, obvious intent, no magic numbers

## What Makes Code Elite

Focus on:
- **Result Types**: Use discriminated union patterns for operations that can fail. Return success/failure explicitly rather than throwing or returning null.
- **Custom Errors**: Create specific error classes with codes and context. Generic `Error` is banned for business logic.
- **Boundary Validation**: Validate all external inputs at system boundaries. Trust internal code, distrust external data.
- **Exhaustive Handling**: Switch statements should be exhaustive. All code paths must be handled explicitly.

## Anti-Patterns (Banned)

NEVER write code containing:
- `// TODO:` or `// FIXME:` comments
- `throw new Error('Not implemented')`
- `as any` or `as unknown` type assertions without guards
- Empty catch blocks or silent error swallowing
- `console.log` as error handling
- Functions that return `null` for errors instead of Result types
- Implicit return types on public functions
- Magic numbers without named constants

## Completeness Standard

If you write a function signature, implement it fully. If you cannot implement something, do not include it. Partial implementations are worse than no implementation because they create false confidence.

Every function must:
- Handle all input variations
- Return meaningful results for all code paths
- Log errors with context before re-throwing
- Clean up resources in finally blocks

## The Test

Before considering code complete, ask:
- "Would this crash if given unexpected input?"
- "Are there any code paths that silently fail?"
- "Could a future developer understand this without asking me?"
- "Does every error provide enough context to debug?"

If any answer is uncertain, the code is not elite.
