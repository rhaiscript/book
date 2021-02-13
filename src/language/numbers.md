Numbers
=======

{{#include ../links.md}}

Integer numbers follow C-style format with support for decimal, binary ('`0b`'), octal ('`0o`') and hex ('`0x`') notations.

The default system integer type (also aliased to `INT`) is `i64`. It can be turned into `i32` via the [`only_i32`] feature.

Floating-point numbers are also supported if not disabled with [`no_float`]. The default system floating-point type is `i64`
(also aliased to `FLOAT`). It can be turned into `f32` via the [`f32_float`] feature.

When rounding errors cannot be accepted, such as in financial calculations, the [`decimal`] feature
turns on support for the [`Decimal`][rust_decimal] type, which is a fixed-precision floating-point
number with precise rounding behavior.

'`_`' separators can be added freely and are ignored within a number &ndash; except at the very beginning or right after
a decimal point ('`.`').

| Format            | Format                    |  Type   | [`no_float`] | [`no_float`] + [`decimal`] |
| ----------------- | ------------------------- | :-----: | :----------: | :------------------------: |
| `123_345`, `-42`  | decimal                   |  `INT`  |              |                            |
| `0o07_76`         | octal                     |  `INT`  |              |                            |
| `0xab_cd_ef`      | hex                       |  `INT`  |              |                            |
| `0b0101_1001`     | binary                    |  `INT`  |              |                            |
| `123_456.789`     | normal floating-point     | `FLOAT` |     n/a      | [`Decimal`][rust_decimal]  |
| `-42.`            | ending with decimal point | `FLOAT` |     n/a      | [`Decimal`][rust_decimal]  |
| `123_456.789e-10` | _scientific notation_     | `FLOAT` |     n/a      |            n/a             |


Floating-Point vs. Decimal
--------------------------

[`Decimal`][rust_decimal] (enabled via the [`decimal`] feature) represents a fixed-precision
floating-point number which is popular with financial calculations and other usage scenarios where
round-off errors are not acceptable.

For most situations, the standard floating-point number type `FLOAT` (`f64` or `f32` with
`f32_float`) is enough and is faster than `Decimal`.

It is possible to use both `FLOAT` and `Decimal` together with just the [`decimal`] feature.
Use [`parse_decimal`] or [`to_decimal`] to create a `Decimal` value.


Use `no_float` and `decimal` Together
------------------------------------

If both [`no_float`] and [`decimal`] features are turned on, however, [`Decimal`][rust_decimal]
_replaces_ the standard floating-point type.

In other words, floating-point number literals in scripts will parse to [`Decimal`][rust_decimal] values.
