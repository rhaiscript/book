Numeric Functions
================

{{#include ../links.md}}

Integer Functions
----------------

The following standard functions (defined in the [`ArithmeticPackage`][packages] but excluded if
using a [raw `Engine`]) operate on `i8`, `i16`, `i32`, `i64`, `f32`, `f64` and [`Decimal`][rust_decimal] (requires
[`decimal`]) only:

| Function | Description                                                          |
| -------- | -------------------------------------------------------------------- |
| `abs`    | absolute value                                                       |
| `sign`   | returns (`INT`) &minus;1 if negative, &plus;1 if positive, 0 if zero |

The following standard functions (defined in the [`BasicMathPackage`][packages] but excluded if using a [raw `Engine`])
operate on integers only:

| Function     | Not available under | Description                                              |
| ------------ | :-----------------: | -------------------------------------------------------- |
| `to_float`   |    [`no_float`]     | convert the value into `f64` (`f32` under [`f32_float`]) |
| `to_decimal` |   non-[`decimal`]   | convert the value into [`Decimal`][rust_decimal]         |


Floating-Point Functions
-----------------------

The following standard functions (defined in the [`BasicMathPackage`][packages] but excluded if using a [raw `Engine`])
operate on `f64` only (`f32` under [`f32_float`]):

| Category         | Supports Decimal | Functions                                                                                |
| ---------------- | :--------------: | ---------------------------------------------------------------------------------------- |
| Trigonometry     |        no        | `sin`, `cos`, `tan`, `sinh`, `cosh`, `tanh` in radians, `hypot(`_x_`,`_y_`)`             |
| Arc-trigonometry |        no        | `asin`, `acos`, `atan(`_v_`)`, `atan(`_x_`,`_y_`)`, `asinh`, `acosh`, `atanh` in radians |
| Square root      |       yes        | `sqrt`                                                                                   |
| Exponential      |       yes        | `exp` (base _e_)                                                                         |
| Logarithmic      |       yes        | `ln` (base _e_)                                                                          |
| Logarithmic      |        no        | `log(`_x_`)` in base 10, `log(`_x_`,`_base_`)`                                           |
| Rounding         |       yes        | `floor`, `ceiling`, `round`, `int`, `fraction` methods and properties                    |
| Conversion       |       yes        | [`to_int`], [`to_decimal`] (requires [`decimal`])                                        |
| Conversion       |        no        | `to_degrees`, `to_radians`                                                               |
| Testing          |        no        | `is_nan`, `is_finite`, `is_infinite` methods and properties                              |


Decimal Rounding
----------------

The following rounding methods (defined in the [`BasicMathPackage`][packages] but excluded if using a [raw `Engine`])
operate on [`Decimal`][rust_decimal] only, which requires the [`decimal`] feature:

| Rounding type     | Behavior                                    | Methods                                                      |
| ----------------- | ------------------------------------------- | ------------------------------------------------------------ |
| None              |                                             | `floor`, `ceiling`, `int`, `fraction` methods and properties |
| Banker's rounding | round to integer                            | `round` method and property                                  |
| Banker's rounding | round to specified number of decimal points | `round(`_decimal points_`)`                                  |
| Round up          | away from zero                              | `round_up(`_decimal points_`)`                               |
| Round down        | towards zero                                | `round_down(`_decimal points_`)`                             |
| Round half-up     | mid-point away from zero                    | `round_half_up(`_decimal points_`)`                          |
| Round half-down   | mid-point towards zero                      | `round_half_down(`_decimal points_`)`                        |


Parsing Functions
-----------------

The following standard functions (defined in the [`BasicMathPackage`][packages] but excluded if using a [raw `Engine`])
parse numbers:

| Function          | No available under | Description                                         |
| ----------------- | :----------------: | --------------------------------------------------- |
| [`parse_int`]     |                    | converts a [string] to `INT` with an optional radix |
| [`parse_float`]   |    [`no_float`]    | converts a [string] to `FLOAT`                      |
| [`parse_decimal`] |  non-[`decimal`]   | converts a [string] to [`Decimal`][rust_decimal]    |


Constants
---------

The following functions return standard mathematical constants:

| Function | Description               |
| -------- | ------------------------- |
| `PI`     | returns the value of &pi; |
| `E`      | returns the value of _e_  |
