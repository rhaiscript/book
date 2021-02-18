Value Conversions
=================

{{#include ../links.md}}


Convert Between Integer and Floating-Point
-----------------------------------------

The `to_float` function converts a supported number to `FLOAT` (defaults to `f64`).

The `to_int` function converts a supported value to `INT` (`i32` or `i64` depending on [`only_i32`]).

The `to_decimal` function converts an integer to [`Decimal`][rust_decimal] (requires [`decimal`]).
Converting from floating-point numbers to [`Decimal`][rust_decimal] is not supported due to the
possibility of rounding errors.

That's it; for other conversions, register custom conversion functions.

```rust,no_run
let x = 42;                     // 'x' is an integer

let y = x * 100.0;              // works: floating-point and integer can inter-operate

let y = x.to_float() * 100.0;   // works

let z = y.to_int() + x;         // works

let w = z.to_decimal() + x;     // works: Decimal and integer can inter-operate

let c = 'X';                    // character

print("c is '" + c + "' and its code is " + c.to_int());    // prints "c is 'X' and its code is 88"
```


Parse String into Number
------------------------

The `parse_float` function converts a [string] into a `FLOAT` (defaults to `f64`).

The `parse_int` function converts a [string] into an `INT` (`i32` or `i64` depending on [`only_i32`]).
An optional radix (2-36) can be provided to parse the [string] into a number of the specified radix.

The `parse_decimal` function converts a [string] into a [`Decimal`][rust_decimal] (requires [`decimal`]).

```rust,no_run
let x = parse_float("123.4");   // parse as floating-point
x == 123.4;
type_of(x) == "f64";

let x = parse_decimal("123.4"); // parse as Decimal value
type_of(x) == "decimal";

let x = 1234.to_decimal() / 10; // alternate method to create a Decimal value
type_of(x) == "decimal";

let dec = parse_int("42");      // parse as integer
dec == 42;
type_of(dec) == "i64";

let dec = parse_int("42", 10);  // radix = 10 is the default
dec == 42;
type_of(dec) == "i64";

let bin = parse_int("110", 2);  // parse as binary (radix = 2)
bin == 0b110;
type_of(bin) == "i64";

let hex = parse_int("ab", 16);  // parse as hex (radix = 16)
hex == 0xab;
type_of(hex) == "i64";
```
