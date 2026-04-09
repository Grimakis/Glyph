# Glyph Language Specification — V1 Draft

## 1. Language Shape

Glyph is a procedural, struct-based, compiled language. It targets the Intel
8085 processor as found in the TRS-80 Model 100. Programs compile to native
machine-language `.CO` files.

Glyph is **not** object-oriented. There are no classes, inheritance, or
dynamic dispatch. There are no generics or traits in V1.

## 2. Goals

- Readable, minimal syntax for 8-bit systems programming.
- Predictable mapping from source to 8085 assembly.
- Small compiler with a long-term path toward self-hosting.
- Thin runtime using documented Model 100 ROM routines.

## 3. Non-Goals (V1)

- Heap allocation or garbage collection.
- Recursion.
- Floating-point arithmetic in the core language.
- Exceptions or unwinding.
- Full borrow checker or ownership system.
- OOP features of any kind.

## 4. Safety Stance

Glyph V1 is an **unsafe systems language**. Raw pointer arithmetic and
unchecked array access are permitted. In V1, `unsafe` is mainly a marker for
low-level intent, not a strict safety boundary. The compiler does not yet
enforce a safe/unsafe split. `unsafe` exists now so low-level code can be
annotated and the syntax is reserved for future versions, which may attach
stronger rules to it.

## 5. Syntax Overview

Glyph uses curly-brace blocks, semicolon-terminated statements, and a
keyword-driven declaration style. Indentation is not significant.

### 5.1 Top-Level Declarations

| Keyword       | Purpose                          |
|---------------|----------------------------------|
| `const`       | Compile-time constant            |
| `var`         | Module-level mutable variable    |
| `type`        | Type alias or struct definition  |
| `proc`        | Procedure definition             |
| `extern proc` | External (runtime) procedure     |

### 5.2 Local Declarations

| Keyword | Purpose                             |
|---------|-------------------------------------|
| `let`   | Immutable local binding             |
| `var`   | Mutable local variable              |

## 6. Lexical Rules

- Source encoding: ASCII (7-bit).
- Line endings: LF or CR+LF.
- Comments: `//` to end of line.
- Identifiers: `[a-zA-Z_][a-zA-Z0-9_]*`.
- Integer literals: decimal (`123`), hexadecimal (`0xFF`), binary (`0b1010`).
- String literals: double-quoted, null-terminated. Supports `\n`, `\r`,
  `\t`, `\\`, `\0`, `\"`.
- Character literals: single-quoted (`'A'`).
- Boolean literals: `true`, `false`.

## 7. Keywords

```
as        bool      break     const     continue  else
extern    false     i8        i16       if        let
proc      ptr       ram       return    rom       scratch
sizeof    struct    true      type      u8        u16
unsafe    var       void      while
```

## 8. Core Types

| Type   | Size    | Description                  |
|--------|---------|------------------------------|
| `u8`   | 1 byte  | Unsigned 8-bit integer       |
| `i8`   | 1 byte  | Signed 8-bit integer (two's complement) |
| `u16`  | 2 bytes | Unsigned 16-bit integer      |
| `i16`  | 2 bytes | Signed 16-bit integer (two's complement) |
| `bool` | 1 byte  | Boolean (`true` / `false`)   |
| `void` | 0 bytes | No value                     |

Integer semantics: wrapping on overflow. No undefined behavior for overflow.

## 9. Compound Types

### 9.1 Fixed Arrays

```
[N]T
```

`N` must be a compile-time constant. Arrays are value types laid out
contiguously in memory. There are no dynamically-sized arrays.

### 9.2 Pointers

```
ptr<T>
```

A pointer holds a 16-bit address. Pointer arithmetic is permitted. Null
pointers are the programmer's responsibility to avoid.

### 9.3 Structs

```
type Point struct {
    x: i16;
    y: i16;
}
```

Structs are laid out in declaration order with no implicit padding (the
compiler may add explicit padding directives in a future version). Structs
are value types; non-trivial structs are passed by pointer in function calls.

## 10. Storage Classes

| Class     | Meaning                                     |
|-----------|---------------------------------------------|
| `rom`     | Stored in the `.CO` image (read-only data)  |
| `ram`     | Allocated in the writable data segment      |
| `scratch` | Scratch memory (e.g., Model 100 scratch RAM)|

Storage class annotations appear only on top-level `const` and `var`
declarations. Local `let` and local `var` declarations do not take storage
classes.

## 11. `const` / `let` / `var`

- `const` — compile-time constant, usable at top level. Must have a value
  known at compile time.
- `let` — immutable local binding. Assigned once.
- `var` — mutable variable. Usable at top level (module storage) or in
  local scope.

```
const NAME: [storage_class] TYPE = expr;
var NAME: [storage_class] TYPE [= expr];

let NAME: TYPE = expr;
var NAME: TYPE [= expr];
```

```
const COLUMNS: u8 = 40;
var cursor_x: ram u8 = 0;

proc example() -> void {
    let limit: u8 = COLUMNS;
    var i: u8 = 0;
    // ...
}
```

## 12. Procedures

```
proc name(param1: T1, param2: T2) -> ReturnType {
    // body
}
```

- No default arguments.
- No variadic arguments in V1.
- No recursion. The compiler may reject or warn on recursive call graphs.
- `extern proc` declares a procedure supplied by the runtime or linker.

## 13. Statements

| Statement                 | Description                        |
|---------------------------|------------------------------------|
| `let x: T = expr;`       | Immutable local binding            |
| `var x: T = expr;`       | Mutable local declaration          |
| `x = expr;`              | Assignment                         |
| `if cond { ... }`        | Conditional                        |
| `if cond { ... } else { ... }` | Conditional with else branch |
| `while cond { ... }`     | Loop                               |
| `break;`                 | Exit innermost loop                |
| `continue;`              | Next iteration of innermost loop   |
| `return expr;`           | Return from procedure              |
| `return;`                | Return void                        |
| `unsafe { ... }`         | Unsafe block                       |
| `expr;`                  | Expression statement (calls, etc.) |

## 14. Expressions

Expressions include:

- Integer, boolean, character, and string literals.
- Identifiers.
- Binary operators: `+`, `-`, `*`, `/`, `%`, `&`, `|`, `^`, `<<`, `>>`,
  `==`, `!=`, `<`, `>`, `<=`, `>=`, `&&`, `||`.
- Unary operators: `-`, `!`, `~`, `&` (address-of), `*` (dereference).
- Array indexing: `a[i]`.
- Struct field access: `s.field`.
- Procedure calls: `name(args)`.
- Cast: `expr as T`.
- `sizeof(T)` — compile-time size in bytes.
- Parenthesized expressions.

Operator precedence follows conventional C-family ordering. A full precedence
table will be defined when the parser is implemented.

## 15. Casts

```
let wide: u16 = narrow as u16;
```

`as` performs explicit type conversion. Widening (u8 → u16) zero-extends.
Narrowing (u16 → u8) truncates. Signed/unsigned reinterpretation is allowed.
Pointer casts are permitted in V1; `unsafe` blocks may be used to mark
low-level code that relies on them.

## 16. `sizeof`

```
let sz: u16 = sizeof(Point);
```

`sizeof` is a compile-time operator that yields the size in bytes of a type
or expression. The result type is `u16`.

## 17. Memory Model

- No heap. All storage is either static (module-level) or stack-based (local).
- Stack frames are compile-time-known in size (no recursion, no VLAs).
- The stack grows downward.
- Pointers are 16-bit absolute addresses.

## 18. Bounds Policy

V1 does **not** require runtime bounds checking on array accesses.
Out-of-bounds access is undefined behavior in V1. A future optional checked
mode may add compile-time or runtime checks.

## 19. Integer Semantics

- All integer arithmetic wraps on overflow (modular arithmetic).
- Division by zero is undefined in V1 (the compiler may trap or produce
  garbage).
- Shift amounts exceeding the bit width are undefined in V1.
- There is no implicit integer promotion. All operands must match in type,
  or one must be explicitly cast.

## 20. Platform ABI Overview

See `abi.md` for the full V1 ABI specification. Summary:

- Arguments passed left-to-right.
- Small scalars may use registers; excess arguments go on the stack.
- Return values: u8/i8 in a byte register, u16/i16/pointer in a register pair.
- No struct return-by-value in V1.

## 21. BASIC Math Reuse Policy

The Model 100 ROM contains floating-point and BCD math routines used by the
built-in BASIC interpreter. Glyph V1 **does not use** these routines.
Integer math is generated natively by the compiler using 8085 instructions.
Optional ROM-assisted higher math (e.g., 32-bit multiply) may be explored in
future versions.

## 22. Error Handling Model

Glyph V1 has no exceptions, no `try`/`catch`, and no `Result` type in the
core language. Errors are communicated through return values (e.g., sentinel
values or out-parameters). This is intentional for a V1 targeting an 8-bit
system with no OS-level exception handling.

## 23. Future Extensions (Post-V1)

The following may be considered for future versions:

- Optional bounds checking (compile-time and runtime).
- Simple ownership annotations.
- Module / import system.
- Inline assembly.
- 32-bit integer support.
- Fixed-point arithmetic.
- Enum types.
- ROM-assisted math for large operations.
- Graphics and I/O device support.

None of these are in scope for V1.

## 24. Example Program

```glyph
const MSG: rom [6]u8 = "HELLO";

extern proc screen_cls() -> void;
extern proc screen_putc(row: u8, col: u8, ch: u8, rev: bool) -> void;
extern proc key_get_blocking() -> u8;

proc print_text(row: u8, col: u8, text: ptr<u8>) -> void {
    var i: u8 = 0;
    while text[i] != 0 {
        screen_putc(row, col + i, text[i], false);
        i = i + 1;
    }
}

proc main() -> void {
    screen_cls();
    print_text(1, 1, &MSG[0]);
    key_get_blocking();
}
```
