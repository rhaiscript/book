Numbers
=======

{{#include ../links.md}}

Integer numbers follow C-style format with support for decimal, binary ('`0b`'), octal ('`0o`') and hex ('`0x`') notations.

The default system integer type (also aliased to `INT`) is `i64`. It can be turned into `i32` via the [`only_i32`] feature.

Floating-point numbers are also supported if not disabled with [`no_float`]. The default system floating-point type is `i64`
(also aliased to `FLOAT`). It can be turned into `f32` via the [`f32_float`] feature.

'`_`' separators can be added freely and are ignored within a number &ndash; except at the very beginning or right after
a decimal point ('`.`').

| Format            | Type                                    |
| ----------------- | --------------------------------------- |
| `123_345`, `-42`  | `INT` in decimal                        |
| `0o07_76`         | `INT` in octal                          |
| `0xab_cd_ef`      | `INT` in hex                            |
| `0b0101_1001`     | `INT` in binary                         |
| `123_456.789`     | `FLOAT` in normal floating-point format |
| `-42.`            | `FLOAT` ending with decimal point       |
| `123_456.789e-10` | `FLOAT` in _scientific notation_        |
