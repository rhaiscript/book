Value Conversions
=================

{{#include ../links.md}}


Convert Between Integer and Floating-Point
-----------------------------------------

The `to_float` function converts a supported number to `FLOAT` (defaults to `f64`).

The `to_int` function converts a supported number to `INT` (`i32` or `i64` depending on [`only_i32`]).

That's it; for other conversions, register custom conversion functions.

```rust
let x = 42;

let y = x * 100.0;              // <- error: cannot multiply i64 with f64

let y = x.to_float() * 100.0;   // works

let z = y.to_int() + x;         // works

let c = 'X';                    // character

print("c is '" + c + "' and its code is " + c.to_int());    // prints "c is 'X' and its code is 88"
```


Parse String into Number
------------------------

The `parse_float` function converts a [string] into a `FLOAT` (defaults to `f64`).

The `parse_int` function converts a [string] into an `INT` (`i32` or `i64` depending on [`only_i32`]).
An optional radix (2-36) can be provided to parse the [string] into a number of the specified radix.

```rust
let x = parse_float("123.4");   // parse as floating-point
x == 123.4;
type_of(x) == "f64";

let dec = parse_int("42");      // parse as decimal
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
