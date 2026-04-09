# Glyph Runtime Philosophy

This document describes how the Glyph runtime layer relates to the compiler
and the TRS-80 Model 100 hardware.

## 1. Thin Runtime

The Glyph runtime is intentionally **thin**. It provides a small set of
wrapper procedures that give Glyph programs access to Model 100 hardware
through documented ROM entry points. The runtime is not a standard library —
it is a platform abstraction layer.

## 2. Compiler Independence from ROM

The **compiler itself** does not depend on ROM routines. Code generation for
arithmetic, control flow, and memory access uses native 8085 instructions.
The compiler can produce a valid `.CO` binary that never calls the ROM.

The **runtime/platform layer** is what calls into ROM. Programs opt in to
ROM services by declaring `extern proc` bindings to runtime wrappers.

## 3. Documented ROM Routines First

The first milestone targets only **documented** ROM entry points — those
published in the TRS-80 Model 100 Technical Reference Manual and widely
verified by the community.

Specifically, milestone 1 **avoids**:

- Undocumented LCD controller tricks.
- Direct manipulation of BASIC interpreter internal state.
- Reliance on RAM locations whose layout varies between ROM versions.

If a useful routine exists in ROM but is undocumented, it stays out of the
first runtime. It can be added later once behavior is verified.

## 4. Integer Math: Generated Natively

All integer arithmetic (8-bit and 16-bit add, subtract, multiply, divide,
shift, compare) is generated as native 8085 assembly by the compiler. The
compiler does **not** call into the BASIC interpreter's math routines.

Rationale:

- The BASIC math routines operate on floating-point or BCD formats, which
  do not match Glyph's integer types.
- Calling conventions for internal BASIC routines are undocumented and
  fragile.
- Native integer math on the 8085 is straightforward and predictable.

## 5. Optional Future ROM-Assisted Math

In future versions, the runtime **may** offer optional wrappers for
ROM-based math operations (e.g., if the Model 100 ROM exposes integer
multiply or divide routines that are faster than software implementations).
These would be:

- Clearly documented.
- Opt-in via explicit `extern proc` declarations.
- Never used implicitly by the compiler.

## 6. Runtime Surface Area

The initial runtime covers:

- **Text screen**: clear, cursor, put character, reverse video, erase.
- **Keyboard**: blocking read, poll, pending check.
- **Clock**: time, date, day-of-week.
- **System**: exit to menu, break check.

See `rom-api.md` for the full list of V1 runtime procedures.

### What Is Not in Milestone 1

- Graphics (pixel plotting, line drawing).
- Printer output.
- Serial / RS-232 communication.
- Cassette tape I/O.
- Sound.
- File system access.
- BASIC interpreter interoperation.

These can be added as later runtime extensions once the core toolchain is
stable.
