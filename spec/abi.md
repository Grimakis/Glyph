# Glyph V1 ABI Specification

This document describes the calling convention and binary interface for
Glyph V1 targeting the Intel 8085 (TRS-80 Model 100).

## 1. Argument Passing

- Arguments are evaluated and passed **left-to-right**.
- Small scalar values (u8, i8, bool) may be passed in registers when
  registers are available.
- u16, i16, and pointer values may be passed in register pairs when
  available.
- When registers are exhausted, remaining arguments are pushed onto the
  stack in left-to-right order.
- Non-trivial structs (larger than 2 bytes) are always passed **by pointer**.

The exact register assignment scheme will be finalized when the backend is
implemented. The initial design reserves:

| Slot | Register(s) | Eligible types              |
|------|-------------|-----------------------------|
| 1    | A           | u8, i8, bool                |
| 2    | HL          | u16, i16, ptr               |
| 3+   | stack       | any                         |

This is a starting point; the backend may adjust register allocation.

## 2. Return Values

| Return type        | Location              |
|--------------------|-----------------------|
| `u8`, `i8`, `bool` | A register           |
| `u16`, `i16`, `ptr` | HL register pair    |
| `void`             | (nothing)             |
| structs            | Not returned by value in V1. Use an out-pointer parameter. |

## 3. Stack Layout

- The stack grows **downward** (high addresses to low addresses), following
  the 8085 convention.
- The stack pointer (SP) points to the last pushed byte.
- Stack frames are **compile-time-known** in size. Because Glyph V1 forbids
  recursion, the compiler can statically compute the maximum stack depth for
  any call graph.

### Stack Frame Structure

```
High address
+-----------------------+
| caller's frame        |
+-----------------------+
| return address (2 B)  |  ← pushed by CALL
+-----------------------+
| saved registers       |  ← callee-saved, if any
+-----------------------+
| local variables       |  ← fixed size, known at compile time
+-----------------------+
| outgoing arguments    |  ← for calls made by this procedure
+-----------------------+  ← SP during procedure body
Low address
```

## 4. Callee-Saved Registers

The set of callee-saved registers will be defined when the backend is
implemented. The initial intent is:

- **Caller-saved**: A, flags.
- **Callee-saved**: BC, DE (tentative).
- **HL**: used for return values and scratch; caller-saved.

## 5. Alignment

The 8085 has no alignment requirements. All values are byte-aligned.
Struct fields are laid out in declaration order with no implicit padding.

## 6. No Heap

Glyph V1 has no heap allocator. All memory is either:

- **Static** — module-level `const` and `var` declarations, placed in the
  `.CO` image or writable data segment.
- **Stack** — local `let` and `var` declarations within procedures.
- **Scratch** — optionally mapped to Model 100 scratch RAM areas.

## 7. Name Mangling

V1 uses a simple name-mangling scheme (or no mangling at all) since there
are no namespaces or overloading. Procedure names map directly to assembly
labels. This may change if a module system is added later.
