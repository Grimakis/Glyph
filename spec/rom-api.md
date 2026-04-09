# Glyph ROM API — V1 Surface

This document defines the initial set of runtime procedures that wrap
TRS-80 Model 100 ROM entry points. These are the procedures available to
Glyph programs via `extern proc` declarations.

## Scope

Milestone 1 focuses on:

- Text screen output
- Keyboard input
- System clock
- System exit

Graphics, printer, serial, cassette, and file I/O are **deferred** to later
milestones.

BASIC internal math helpers are **not** part of the V1 runtime.

---

## Screen Routines

### `screen_cls`

```glyph
extern proc screen_cls() -> void;
```

Clear the LCD screen and reset the cursor to the home position.

### `screen_set_cursor`

```glyph
extern proc screen_set_cursor(row: u8, col: u8) -> void;
```

Move the text cursor to the given row and column. Rows and columns are
1-based.

### `screen_putc`

```glyph
extern proc screen_putc(row: u8, col: u8, ch: u8, rev: bool) -> void;
```

Write a single character at the given screen position. If `rev` is `true`,
the character is displayed in reverse video.

### `screen_lock`

```glyph
extern proc screen_lock() -> void;
```

Disable automatic LCD refresh. Use before bulk screen updates to avoid
flicker.

### `screen_unlock`

```glyph
extern proc screen_unlock() -> void;
```

Re-enable automatic LCD refresh and force an immediate update.

### `screen_reverse_on`

```glyph
extern proc screen_reverse_on() -> void;
```

Enable reverse video mode for subsequent character output through ROM
routines.

### `screen_reverse_off`

```glyph
extern proc screen_reverse_off() -> void;
```

Disable reverse video mode.

### `screen_erase_to_eol`

```glyph
extern proc screen_erase_to_eol() -> void;
```

Erase from the current cursor position to the end of the line.

---

## Keyboard Routines

### `key_get_blocking`

```glyph
extern proc key_get_blocking() -> u8;
```

Wait for a keypress and return the ASCII code of the key pressed.

### `key_poll`

```glyph
extern proc key_poll() -> u8;
```

Check for a keypress without blocking. Returns the ASCII code if a key is
available, or `0` if no key is pending.

### `key_pending`

```glyph
extern proc key_pending() -> bool;
```

Return `true` if a keypress is available in the keyboard buffer.

---

## System Routines

### `break_check`

```glyph
extern proc break_check() -> bool;
```

Return `true` if the user has pressed the BREAK key combination. Programs
should poll this periodically to allow graceful interruption.

### `system_exit_to_menu`

```glyph
extern proc system_exit_to_menu() -> void;
```

Exit the `.CO` program and return to the Model 100 main menu. This does not
return.

---

## Clock Routines

### `clock_time`

```glyph
extern proc clock_time(buf: ptr<u8>) -> void;
```

Write the current time as an 8-byte ASCII string (`HH:MM:SS`) into `buf`.
The buffer must be at least 9 bytes (including null terminator).

### `clock_date`

```glyph
extern proc clock_date(buf: ptr<u8>) -> void;
```

Write the current date as an 8-byte ASCII string (`MM/DD/YY`) into `buf`.
The buffer must be at least 9 bytes (including null terminator).

### `clock_day`

```glyph
extern proc clock_day() -> u8;
```

Return the day of the week as a number: 0 = Sunday, 1 = Monday, ...,
6 = Saturday.

---

## Future Extensions (Not in V1)

The following ROM areas may be wrapped in later milestones:

- **Graphics**: pixel set/clear, line draw, screen copy.
- **Printer**: character output, line feed, status.
- **Serial (RS-232)**: open, close, read, write, status.
- **Cassette**: open, close, read, write.
- **Sound**: beep, tone generation.
- **File system**: BASIC file open/close/read/write (if viable without
  deep BASIC interpreter dependency).

These will be added only after the core compiler and runtime are stable.
