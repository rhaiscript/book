Numeric Functions
=================

{{#title Numeric Functions}}

Integer Functions
-----------------

The following standard functions operate on integers only.

| Function                      | Description                                                      |
| ----------------------------- | ---------------------------------------------------------------- |
| `is_odd` method and property  | returns `true` if the value is an odd number, otherwise `false`  |
| `is_even` method and property | returns `true` if the value is an even number, otherwise `false` |
| `min`                         | returns the smaller of two numbers, the first number if equal    |
| `max`                         | returns the larger of two numbers, the first number if equal     |
| `to_float`                    | convert the value into `f64` (`f32` under 32-bit)                |
| `to_decimal`                  | convert the value into decimal                                   |


Signed Numeric Functions
------------------------

The following standard functions operate on signed numbers (including floating-point and decimal) only.

| Function                      | Description                                            |
| ----------------------------- | ------------------------------------------------------ |
| `abs`                         | absolute value                                         |
| `sign`                        | returns −1 if negative, &plus;1 if positive, 0 if zero |
| `is_zero` method and property | returns `true` if the value is zero, otherwise `false` |


Floating-Point Functions
------------------------

The following standard functions operate on floating-point and decimal numbers only.

| Category         | Decimal? | Functions                                                                                |
| ---------------- | :------: | ---------------------------------------------------------------------------------------- |
| Trigonometry     |   yes    | `sin`, `cos`, `tan`                                                                      |
| Trigonometry     |    no    | `sinh`, `cosh`, `tanh` in radians, `hypot(`_x_`,`_y_`)`                                  |
| Arc-trigonometry |    no    | `asin`, `acos`, `atan(`_v_`)`, `atan(`_x_`,`_y_`)`, `asinh`, `acosh`, `atanh` in radians |
| Square root      |   yes    | `sqrt`                                                                                   |
| Exponential      |   yes    | `exp` (base _e_)                                                                         |
| Logarithmic      |   yes    | `ln` (base _e_), `log` (base 10)                                                         |
| Logarithmic      |    no    | `log(`_x_`,`_base_`)`                                                                    |
| Rounding         |   yes    | `floor`, `ceiling`, `round`, `int`, `fraction` methods and properties                    |
| Conversion       |   yes    | [`to_int`](convert.md), [`to_decimal`](convert.md), [`to_float`](convert.md)             |
| Conversion       |    no    | `to_degrees`, `to_radians`                                                               |
| Comparison       |   yes    | `min`, `max` (also inter-operates with integers)                                         |
| Testing          |    no    | `is_nan`, `is_finite`, `is_infinite` methods and properties                              |


Decimal Rounding Functions
--------------------------

The following rounding methods operate on decimal numbers only.

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

The following standard functions parse numbers.

| Function                      | Description                                                             |
| ----------------------------- | ----------------------------------------------------------------------- |
| [`parse_int`](convert.md)     | converts a [string](strings-chars.md) to integer with an optional radix |
| [`parse_float`](convert.md)   | converts a [string](strings-chars.md) to floating-point                 |
| [`parse_decimal`](convert.md) | converts a [string](strings-chars.md) to decimal                        |


Formatting Functions
--------------------

The following standard functions convert integer numbers into a [string](strings-chars.md) of hex,
octal or binary representations.

| Function                  | Description                          |
| ------------------------- | ------------------------------------ |
| [`to_binary`](convert.md) | converts an integer number to binary |
| [`to_octal`](convert.md)  | converts an integer number to octal  |
| [`to_hex`](convert.md)    | converts an integer number to hex    |


Floating-point Constants
------------------------

The following functions return standard mathematical constants.

| Function | Description               |
| -------- | ------------------------- |
| `PI`     | returns the value of &pi; |
| `E`      | returns the value of _e_  |
