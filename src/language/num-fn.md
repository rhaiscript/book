Numeric Functions
================

{{#include ../links.md}}

Integer Functions
----------------

The following standard functions (defined in the [`ArithmeticPackage`][packages] but excluded if
using a [raw `Engine`]) operate on `i8`, `i16`, `i32`, `i64`, `f32`, `f64` and [`Decimal`][rust_decimal] (requires
[`decimal`]) only:

| Function | Description                                                             |
| -------- | ----------------------------------------------------------------------- |
| `abs`    | absolute value                                                          |
| `sign`   | returns -1 (`INT`) if the number is negative, +1 if positive, 0 if zero |

The following standard functions (defined in the [`BasicMathPackage`][packages] but excluded if using a [raw `Engine`])
operate on integer or [`Decimal`][rust_decimal] (requires [`decimal`]) values:

| Function   | Not available under | Description                                                 |
| ---------- | :-----------------: | ----------------------------------------------------------- |
| `to_float` |    [`no_float`]     | convert the value into an `f64` (`f32` under [`f32_float`]) |


Floating-Point Functions
-----------------------

The following standard functions (defined in the [`BasicMathPackage`][packages] but excluded if using a [raw `Engine`])
operate on `f64` only (`f32` under [`f32_float`]):

| Category         | Functions                                                                   |
| ---------------- | --------------------------------------------------------------------------- |
| Trigonometry     | `sin`, `cos`, `tan`, `sinh`, `cosh`, `tanh` in radians                      |
| Arc-trigonometry | `asin`, `acos`, `atan`, `asinh`, `acosh`, `atanh` in radians                |
| Square root      | `sqrt`                                                                      |
| Exponential      | `exp` (base _e_)                                                            |
| Logarithmic      | `ln` (base _e_), `log10` (base 10), `log` (any base)                        |
| Rounding         | `floor`, `ceiling`, `round`, `int`, `fraction` methods and properties       |
| Conversion       | [`to_int`], `to_decimal` (requires [`decimal`]), `to_degrees`, `to_radians` |
| Testing          | `is_nan`, `is_finite`, `is_infinite` methods and properties                 |


Decimal Rounding
----------------

The following rounding methods (defined in the [`BasicMathPackage`][packages] but excluded if using a [raw `Engine`])
operate on [`Decimal`][rust_decimal] only, which requires the [`decimal`] feature:

| Rounding type     | Methods                                                      |
| ----------------- | ------------------------------------------------------------ |
| General           | `floor`, `ceiling`, `int`, `fraction` methods and properties |
| Banker's rounding | `round` method and property                                  |
| Banker's rounding | `round(decimal_points)`                                      |
| Round up          | `round_up(decimal_points)`                                   |
| Round down        | `round_down(decimal_points)`                                 |
| Round half-up     | `round_half_up(decimal_points)`                              |
| Round half-down   | `round_half_down(decimal_points)`                            |


Parsing Functions
-----------------

The following standard functions (defined in the [`BasicMathPackage`][packages] but excluded if using a [raw `Engine`])
parse numbers:

| Function          | No available under | Description                                         |
| ----------------- | :----------------: | --------------------------------------------------- |
| [`parse_int`]     |                    | converts a [string] to `INT` with an optional radix |
| [`parse_float`]   |    [`no_float`]    | converts a [string] to `FLOAT`                      |
| [`parse_decimal`] |  non-[`decimal`]   | converts a [string] to `Decimal`                    |


Constants
---------

The following functions return standard mathematical constants:

| Function | Description               |
| -------- | ------------------------- |
| `PI`     | returns the value of &pi; |
| `E`      | returns the value of _e_  |
