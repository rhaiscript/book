Logic Operators
==============

{{#include ../links.md}}

Comparison Operators
--------------------

| Operator | Description<br/>(`x` _operator_ `y`) | `x`, `y` same type or are numeric | `x`, `y` different types |
| :------: | ------------------------------------ | :-------------------------------: | :----------------------: |
|   `==`   | `x` is equals to `y`                 |       error if not defined        |  `false` if not defined  |
|   `!=`   | `x` is not equals to `y`             |       error if not defined        |  `true` if not defined   |
|   `>`    | `x` is greater than `y`              |       error if not defined        |  `false` if not defined  |
|   `>=`   | `x` is greater than or equals to `y` |       error if not defined        |  `false` if not defined  |
|   `<`    | `x` is less than `y`                 |       error if not defined        |  `false` if not defined  |
|   `<=`   | `x` is less than or equals to `y`    |       error if not defined        |  `false` if not defined  |

Comparison operators between most values of the same type are built in for all [standard types].

Others are defined in the [`LogicPackage`][built-in packages] but excluded if using a [raw `Engine`].


### Floating-point numbers interoperate with integers

Comparing a floating-point number (`FLOAT`) with an integer is also supported.

```rust,no_run
42 == 42.0;         // true

42.0 == 42;         // true

42.0 > 42;          // false

42 >= 42.0;         // true

42.0 < 42;          // false
```

### Decimal numbers interoperate with integers

Comparing a [`Decimal`][rust_decimal] number with an integer is also supported.

```rust,no_run
let d = parse_decimal("42");

42 == d;            // true

d == 42;            // true

d > 42;             // false

42 >= d;            // true

d < 42;             // false
```

### Strings interoperate with characters

Comparing a [string] with a [character] is also supported, with the character first turned into a
[string] before performing the comparison.

```rust,no_run
'x' == "x";         // true

"" < 'a';           // true

'x' > "hello";      // false
```

### Comparing different types defaults to `false`

Comparing two values of _different_ data types defaults to `false` unless the appropriate operator
functions have been registered.

The exception is `!=` (not equals) which defaults to `true`. This is in line with intuition.

```rust,no_run
42 > "42";          // false: i64 cannot be compared with string

42 <= "42";         // false: i64 cannot be compared with string

let ts = new_ts();  // custom type

ts == 42;           // false: different types cannot be compared

ts != 42;           // true: different types cannot be compared

ts == ts;           // error: '==' not defined for the custom type
```

### Safety valve: Comparing different _numeric_ types has no default

Beware that the above default does _NOT_ apply to numeric values of different types
(e.g. comparison between `i64` and `u16`, `i32` and `f64`) &ndash; when multiple numeric types are
used it is too easy to mess up and for subtle errors to creep in.

```rust,no_run
// Assume variable 'x' = 42_u16, 'y' = 42_u16 (both types of u16)

x == y;             // true: '==' operator for u16 is built-in

x == "hello";       // false: different non-numeric operand types default to false

x == 42;            // error: ==(u16, i64) not defined, no default for numeric types

42 == y;            // error: ==(i64, u16) not defined, no default for numeric types
```

### Caution: Beware operators for custom types

Operators are completely separate from each other.  For example:

* `!=` does not equal `!(==)`

* `>` does not equal `!(<=)`

* `<=` does not equal `<` plus `==`

* `<=` does not imply `<`

Therefore, if a [custom type] misses an [operator] definition, it simply raises an error
or returns the default.

This behavior can be counter-intuitive.

```rust,no_run
let ts = new_ts();  // custom type with '<=' and '==' defined

ts <= ts;           // true: '<=' defined

ts < ts;            // error: '<' not defined, even though '<=' is

ts == ts;           // true: '==' defined

ts != ts;           // error: '!=' not defined, even though '==' is
```

It is strongly recommended that, when defining operators for [custom types], always define the full set
of six operators (or at least the `==` and `!=` pair) together.


Boolean operators
-----------------

|     Operator      | Description   | Arity  | Short-Circuits? |
| :---------------: | ------------- | :----: | :-------------: |
|  `!` _(prefix)_   | boolean _NOT_ | unary  |       no        |
|       `&&`        | boolean _AND_ | binary |       yes       |
|        `&`        | boolean _AND_ | binary |       no        |
| <code>\|\|</code> | boolean _OR_  | binary |       yes       |
|  <code>\|</code>  | boolean _OR_  | binary |       no        |

Double boolean operators `&&` and `||` _short-circuit_ &ndash; meaning that the second operand will not be evaluated
if the first one already proves the condition wrong.

Single boolean operators `&` and `|` always evaluate both operands.

```rust,no_run
a() || b();         // b() is not evaluated if a() is true

a() && b();         // b() is not evaluated if a() is false

a() | b();          // both a() and b() are evaluated

a() & b();          // both a() and b() are evaluated
```

All boolean operators are [built in][built-in operators] for the `bool` data type.
