Numeric Functions
=================

{{#include ../links.md}}


Integer Functions
-----------------

The following standard functions are defined.

| Function                      | Not available under |                 Package                  | Description                                                      |
| ----------------------------- | :-----------------: | :--------------------------------------: | ---------------------------------------------------------------- |
| `is_odd` method and property  |                     | [`ArithmeticPackage`][built-in packages] | returns `true` if the value is an odd number, otherwise `false`  |
| `is_even` method and property |                     | [`ArithmeticPackage`][built-in packages] | returns `true` if the value is an even number, otherwise `false` |
| `min`                         |                     |   [`LogicPackage`][built-in packages]    | returns the smaller of two numbers                               |
| `max`                         |                     |   [`LogicPackage`][built-in packages]    | returns the larger of two numbers                                |
| `to_float`                    |    [`no_float`]     | [`BasicMathPackage`][built-in packages]  | convert the value into `f64` (`f32` under [`f32_float`])         |
| `to_decimal`                  |   non-[`decimal`]   | [`BasicMathPackage`][built-in packages]  | convert the value into [`Decimal`][rust_decimal]                 |


Signed Numeric Functions
------------------------

The following standard functions are defined in the [`ArithmeticPackage`][built-in packages]
(excluded when using a [raw `Engine`]) and operate on `i8`, `i16`, `i32`, `i64`, `f32`, `f64` and
[`Decimal`][rust_decimal] (requires [`decimal`]) only.

| Function                      | Description                                                    |
| ----------------------------- | -------------------------------------------------------------- |
| `abs`                         | absolute value                                                 |
| `sign`                        | returns (`INT`) −1 if negative, &plus;1 if positive, 0 if zero |
| `is_zero` method and property | returns `true` if the value is zero, otherwise `false`         |


Floating-Point Functions
------------------------

The following standard functions are defined in the [`BasicMathPackage`][built-in packages]
(excluded when using a [raw `Engine`]) and operate on `f64` (`f32` under [`f32_float`]) and
[`Decimal`][rust_decimal] (requires [`decimal`]) only.

| Category         | Supports `Decimal` | Functions                                                                                |
| ---------------- | :----------------: | ---------------------------------------------------------------------------------------- |
| Trigonometry     |        yes         | `sin`, `cos`, `tan`                                                                      |
| Trigonometry     |       **no**       | `sinh`, `cosh`, `tanh` in radians, `hypot(`_x_`,`_y_`)`                                  |
| Arc-trigonometry |       **no**       | `asin`, `acos`, `atan(`_v_`)`, `atan(`_x_`,`_y_`)`, `asinh`, `acosh`, `atanh` in radians |
| Square root      |        yes         | `sqrt`                                                                                   |
| Exponential      |        yes         | `exp` (base _e_)                                                                         |
| Logarithmic      |        yes         | `ln` (base _e_)                                                                          |
| Logarithmic      |        yes         | `log` (base 10)                                                                          |
| Logarithmic      |       **no**       | `log(`_x_`,`_base_`)`                                                                    |
| Rounding         |        yes         | `floor`, `ceiling`, `round`, `int`, `fraction` methods and properties                    |
| Conversion       |        yes         | [`to_int`], [`to_decimal`] (requires [`decimal`]), [`to_float`] (not under [`no_float`]) |
| Conversion       |       **no**       | `to_degrees`, `to_radians`                                                               |
| Comparison       |        yes         | `min`, `max` (also inter-operates with integers)                                         |
| Testing          |       **no**       | `is_nan`, `is_finite`, `is_infinite` methods and properties                              |


Decimal Rounding Functions
--------------------------

The following rounding methods are defined in the [`BasicMathPackage`][built-in packages]
(excluded when using a [raw `Engine`]) and operate on [`Decimal`][rust_decimal] only,
which requires the [`decimal`] feature.

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

The following standard functions are defined in the [`BasicMathPackage`][built-in packages]
(excluded when using a [raw `Engine`]) to parse numbers.

| Function          |        No available under        | Description                                                                                   |
| ----------------- | :------------------------------: | --------------------------------------------------------------------------------------------- |
| [`parse_int`]     |                                  | converts a [string] to `INT` with an optional radix                                           |
| [`parse_float`]   | [`no_float`] and non-[`decimal`] | converts a [string] to `FLOAT` ([`Decimal`][rust_decimal] under [`no_float`] and [`decimal`]) |
| [`parse_decimal`] |         non-[`decimal`]          | converts a [string] to [`Decimal`][rust_decimal]                                              |


Formatting Functions
--------------------

The following standard functions are defined in the [`BasicStringPackage`][built-in packages]
(excluded when using a [raw `Engine`]) to convert integer numbers into a [string] of hex, octal
or binary representations.

| Function      | Description                          |
| ------------- | ------------------------------------ |
| [`to_binary`] | converts an integer number to binary |
| [`to_octal`]  | converts an integer number to octal  |
| [`to_hex`]    | converts an integer number to hex    |

These formatting functions are defined for all available integer numbers &ndash; i.e. `INT`, `u8`,
`i8`, `u16`, `i16`, `u32`, `i32`, `u64`, `i64`, `u128` and `i128` unless disabled by feature flags.


Floating-point Constants
------------------------

The following functions return standard mathematical constants.

| Function | Description               |
| -------- | ------------------------- |
| `PI`     | returns the value of &pi; |
| `E`      | returns the value of _e_  |


Numerical Functions for Scientific Computing
--------------------------------------------

Check out the [`rhai-sci`] crate for more numerical functions.
