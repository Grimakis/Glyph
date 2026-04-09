# Glyph

**A systems language for the TRS-80 Model 100**

## What Glyph Is

Glyph is a small, procedural, compiled language targeting the Intel 8085-based
TRS-80 Model 100 portable computer. It produces native `.CO` (machine-language)
files that run directly from the Model 100 menu.

Glyph is struct-based and integer-first. It is **not** object-oriented, not a
C clone, and not a Rust clone. V1 is an unsafe systems language designed for
predictable code generation on a very constrained 8-bit platform.

## Project Goals

- Provide a readable, minimal language for Model 100 development.
- Generate tight 8085 assembly from a small compiler with a long-term path
  toward self-hosting.
- Use documented ROM routines through a thin platform runtime layer.
- Keep the entire toolchain understandable by a single developer.

## Non-Goals for V1

- No heap allocation or garbage collection.
- No recursion (stack frames are compile-time-known).
- No floating-point in the core language.
- No exceptions.
- No classes, inheritance, or dynamic dispatch.
- No full borrow checker or advanced memory-safety machinery.
- No generics or traits.

## Current Status

**Phase 1 — Repository skeleton and specification drafts.**

The compiler does not exist yet. This repository contains language
specifications, ABI documentation, runtime design notes, and example programs.

## Repository Layout

```
README.md          Project overview (this file)
AGENTS.md          Guidance for coding agents
spec/
  language.md      Language specification
  grammar.ebnf     EBNF grammar draft
  abi.md           V1 calling convention and ABI
  runtime.md       Runtime philosophy and design
  rom-api.md       Model 100 ROM API surface
examples/
  hello.glyph     Tiny example program
```

## Near-Term Roadmap

1. Repo skeleton and spec documents ← *current*
2. Lexer
3. Parser
4. AST definition
5. Semantic analysis
6. Const data emission
7. Simple 8085 assembly backend
8. Model 100 ROM wrapper runtime
9. `.CO` packaging
