# AGENTS.md — Guidance for Coding Agents

This document describes priorities, constraints, and milestone order for any
agent working on the Glyph project.

## Project Priorities

1. **Keep the language and compiler small.** Glyph targets an 8-bit machine
   with 32 KB of RAM. Every feature must justify its weight.
2. **Prefer explicit behavior.** No hidden allocations, no implicit
   conversions beyond widening, no magic.
3. **Optimize for predictable native code generation.** The programmer should
   be able to reason about the assembly the compiler will emit.
4. **Keep the runtime thin and Model-100-specific.** The runtime wraps
   documented ROM entry points — nothing more.
5. **Avoid feature creep before the backend works.** Get a minimal program
   compiling to a `.CO` before adding language surface area.

## V1 Constraints

The following features are **explicitly excluded** from V1:

- No heap allocation.
- No garbage collection.
- No recursion (all stack frames are compile-time-known).
- No floating-point in the core language.
- No exceptions or unwinding.
- No full borrow checker or ownership system.
- No OOP features (no classes, inheritance, dynamic dispatch, traits).
- No generics.

Do **not** add generics, traits, or memory-safety machinery in V1.

## Milestone Order

1. Repository skeleton and spec documents
2. Lexer
3. Parser
4. AST definition
5. Semantic analysis
6. Const data emission
7. Simple 8085 assembly backend
8. Model 100 ROM wrapper runtime
9. `.CO` packaging

Work on milestones sequentially. Each milestone should be small, testable,
and reviewable before moving to the next.

## Coding Guidelines

- Write clear, minimal code. Favor readability over cleverness.
- Keep modules small and self-contained.
- Add tests for every compiler phase before moving on.
- Use documented ROM routines only; avoid undocumented LCD tricks or BASIC
  interpreter internals.
- Integer math is generated natively by the compiler — do not call into BASIC
  math helpers.
