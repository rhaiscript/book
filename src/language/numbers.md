Numbers
=======

{{#include ../links.md}}


Integers
--------

Integer numbers follow C-style format with support for decimal, binary (`0b`), octal (`0o`) and hex (`0x`) notations.

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

`_` separators can be added freely and are ignored within a number &ndash; except at the very beginning or right after
a decimal point (`.`).

| Sample             | Format                               |  Type   |  [`no_float`]  | [`no_float`] + [`decimal`] |
| ------------------ | ------------------------------------ | :-----: | :------------: | :------------------------: |
| `_123`             | _syntax error (improper separator)_  |         |                |                            |
| `123_345`, `-42`   | decimal                              |  `INT`  |     `INT`      |           `INT`            |
| `0o07_76`          | octal                                |  `INT`  |     `INT`      |           `INT`            |
| `0xab_cd_ef`       | hex                                  |  `INT`  |     `INT`      |           `INT`            |
| `0b0101_1001`      | binary                               |  `INT`  |     `INT`      |           `INT`            |
| `123._456`         | _syntax error (improper separator)_  |         |                |                            |
| `123_456.78_9`     | normal floating-point                | `FLOAT` | _syntax error_ | [`Decimal`][rust_decimal]  |
| `-42.`             | ending with decimal point            | `FLOAT` | _syntax error_ | [`Decimal`][rust_decimal]  |
| `123_456_.789e-10` | _scientific notation_                | `FLOAT` | _syntax error_ | [`Decimal`][rust_decimal]  |
| `.456`             | _syntax error (missing leading `0`)_ |         |                |                            |
| `123.456e_10`      | _syntax error (improper separator)_  |         |                |                            |
| `123.e-10`         | _syntax error (missing decimal `0`)_ |         |                |                            |
| `.456e-10`         | _syntax error (missing leading `0`)_ |         |                |                            |


Warning &ndash; No Implicit Type Conversions
-------------------------------------------

Unlike most C-like languages, Rhai does _not_ provide implicit type conversions between different
numeric types.

For example, a `u8` is never implicitly converted to `i64` when used as a parameter in a function
call or as a comparison operand.  `f32` is never implicitly converted to `f64`.

This is exactly the same as Rust where all numeric types are distinct.  Rhai is written in Rust afterall.

Therefore, care must be taken especially with regards to integer variables pushed inside a custom [`Scope`]
that they are of the intended type.

It is extremely easy to mess up numeric types since the Rust default integer type is `i32` while for
Rhai it is `i64` (without [`only_i32`]).

```rust
use rhai::{Engine, Scope, INT};

let engine = Engine::new();

let mut scope = Scope::new();

scope.push("r", 42);            // 'r' is i32 (Rust default integer type)
scope.push("x", 42_u8);         // 'x' is u8
scope.push("y", 42_i64);        // 'y' is i64
scope.push("z", 42 as INT);     // 'z' is i64 (or i32 under 'only_i32')
scope.push("f", 42.0_f32);      // 'f' is f32

// Rhai integers are i64 (i32 under 'only_i32')
engine.eval::<String>("type_of(42)")? == "i64";

// false - i32 is never equal to i64
engine.eval_with_scope::<bool>(&mut scope, "r == 42")?;

// false - u8 is never equal to i64
engine.eval_with_scope::<bool>(&mut scope, "x == 42")?;

// true - i64 is equal to i64
engine.eval_with_scope::<bool>(&mut scope, "y == 42")?;

// true - INT is i64
engine.eval_with_scope::<bool>(&mut scope, "z == 42")?;

// false - f32 is never equal to f64
engine.eval_with_scope::<bool>(&mut scope, "f == 42.0")?;
```


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
