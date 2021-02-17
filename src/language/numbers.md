Numbers
=======

{{#include ../links.md}}


Integers
--------

Integer numbers follow C-style format with support for decimal, binary ('`0b`'), octal ('`0o`') and hex ('`0x`') notations.

The default system integer type (also aliased to `INT`) is `i64`. It can be turned into `i32` via the [`only_i32`] feature.


Floating-Point Numbers
----------------------

Floating-point numbers are also supported if not disabled with [`no_float`].

Both decimal and scientific notations can be used.

The default system floating-point type is `i64` (also aliased to `FLOAT`).
It can be turned into `f32` via the [`f32_float`] feature.


`Decimal` Numbers
-----------------

When rounding errors cannot be accepted, such as in financial calculations, the [`decimal`] feature
turns on support for the [`Decimal`][rust_decimal] type, which is a fixed-precision floating-point
number with no rounding errors.


Number Literals
---------------

'`_`' separators can be added freely and are ignored within a number &ndash; except at the very beginning or right after
a decimal point ('`.`').

| Sample            | Format                    |  Type   |  [`no_float`]  | [`no_float`] + [`decimal`] |
| ----------------- | ------------------------- | :-----: | :------------: | :------------------------: |
| `_123`            | _syntax error_            |         |                |                            |
| `123_345`, `-42`  | decimal                   |  `INT`  |     `INT`      |           `INT`            |
| `0o07_76`         | octal                     |  `INT`  |     `INT`      |           `INT`            |
| `0xab_cd_ef`      | hex                       |  `INT`  |     `INT`      |           `INT`            |
| `0b0101_1001`     | binary                    |  `INT`  |     `INT`      |           `INT`            |
| `123._456`        | _syntax error_            |         |                |                            |
| `123_456.78_9`    | normal floating-point     | `FLOAT` | _syntax error_ | [`Decimal`][rust_decimal]  |
| `-42.`            | ending with decimal point | `FLOAT` | _syntax error_ | [`Decimal`][rust_decimal]  |
| `123_456.789e-10` | _scientific notation_     | `FLOAT` | _syntax error_ | [`Decimal`][rust_decimal]  |


Floating-Point vs. Decimal
--------------------------

[`Decimal`][rust_decimal] (enabled via the [`decimal`] feature) represents a fixed-precision
floating-point number which is popular with financial calculations and other usage scenarios where
round-off errors are not acceptable.

[`Decimal`][rust_decimal] takes up more space (16 bytes) than a standard `FLOAT` (4-8 bytes) and is
much slower in calculations due to the lack of CPU hardware support. Use it only when necessary.

For most situations, the standard floating-point number type `FLOAT` (`f64` or `f32` with
[`f32_float`]) is enough and is faster than [`Decimal`][rust_decimal].

It is possible to use both `FLOAT` and [`Decimal`][rust_decimal] together with just the [`decimal`] feature
&ndash; use [`parse_decimal`] or [`to_decimal`] to create a [`Decimal`][rust_decimal] value.


Use `no_float` and `decimal` Together
------------------------------------

When both [`no_float`] and [`decimal`] features are turned on, [`Decimal`][rust_decimal] _replaces_
the standard floating-point type.

Floating-point number literals in scripts parse to [`Decimal`][rust_decimal] values.
