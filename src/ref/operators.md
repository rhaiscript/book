Comparison Operators
====================

| Operator | Description<br/>(`x` _operator_ `y`) | `x`, `y` same type or are numeric | `x`, `y` different types |
| :------: | ------------------------------------ | :-------------------------------: | :----------------------: |
|   `==`   | `x` is equals to `y`                 |       error if not defined        |  `false` if not defined  |
|   `!=`   | `x` is not equals to `y`             |       error if not defined        |  `true` if not defined   |
|   `>`    | `x` is greater than `y`              |       error if not defined        |  `false` if not defined  |
|   `>=`   | `x` is greater than or equals to `y` |       error if not defined        |  `false` if not defined  |
|   `<`    | `x` is less than `y`                 |       error if not defined        |  `false` if not defined  |
|   `<=`   | `x` is less than or equals to `y`    |       error if not defined        |  `false` if not defined  |

Comparison operators between most values of the same type are built in for all [standard types](values-and-types.md).


### Floating-point numbers interoperate with integers

Comparing a floating-point number with an integer is also supported.

```rust
42 == 42.0;         // true

42.0 == 42;         // true

42.0 > 42;          // false

42 >= 42.0;         // true

42.0 < 42;          // false
```

### Decimal numbers interoperate with integers

Comparing a decimal number with an integer is also supported.

```rust
let d = parse_decimal("42");

42 == d;            // true

d == 42;            // true

d > 42;             // false

42 >= d;            // true

d < 42;             // false
```

### Strings interoperate with characters

Comparing a [string](strings-chars.md) with a [character](strings-chars.md) is also supported, with
the character first turned into a [string](strings-chars.md) before performing the comparison.

```rust
'x' == "x";         // true

"" < 'a';           // true

'x' > "hello";      // false
```

### Comparing different types defaults to `false`

Comparing two values of _different_ data types defaults to `false` unless the appropriate operator
functions have been registered.

The exception is `!=` (not equals) which defaults to `true`. This is in line with intuition.

```rust
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

```rust
// Assume variable 'x' = 42_u16, 'y' = 42_u16 (both types of u16)

x == y;             // true: '==' operator for u16 is built-in

x == "hello";       // false: different non-numeric operand types default to false

x == 42;            // error: ==(u16, i64) not defined, no default for numeric types

42 == y;            // error: ==(i64, u16) not defined, no default for numeric types
```


Boolean Operators
=================

```admonish note.side

All boolean operators are [built in](../engine/builtin.md) for the `bool` data type.
```

|     Operator      | Description | Arity  | Short-circuits? |
| :---------------: | :---------: | :----: | :-------------: |
|  `!` _(prefix)_   |    _NOT_    | unary  |       no        |
|       `&&`        |    _AND_    | binary |       yes       |
|        `&`        |    _AND_    | binary |       no        |
| <code>\|\|</code> |    _OR_     | binary |       yes       |
|  <code>\|</code>  |    _OR_     | binary |       no        |

Double boolean operators `&&` and `||` _short-circuit_ &ndash; meaning that the second operand will not be evaluated
if the first one already proves the condition wrong.

Single boolean operators `&` and `|` always evaluate both operands.

```rust
a() || b();         // b() is not evaluated if a() is true

a() && b();         // b() is not evaluated if a() is false

a() | b();          // both a() and b() are evaluated

a() & b();          // both a() and b() are evaluated
```


In Operator
===========

```admonish question.side.wide "Trivia"

The `in` operator is simply syntactic sugar for a call to the `contains` function.
```

The `in` operator is used to check for _containment_ &ndash; i.e. whether a particular collection
data type _contains_ a particular item.

```rust
42 in array;

array.contains(42);     // <- the above is equivalent to this
```

### Built-in support for standard data types

|          Data type           |                            Check for                            |
| :--------------------------: | :-------------------------------------------------------------: |
|  Numeric [range](ranges.md)  |                         integer number                          |
|      [Array](arrays.md)      |                         contained item                          |
| [Object map](object-maps.md) |                          property name                          |
|  [String](strings-chars.md)  | [sub-string](strings-chars.md) or [character](strings-chars.md) |

### Examples

```rust
let array = [1, "abc", 42, ()];

42 in array == true;                // check array for item

let map = #{
    foo: 42,
    bar: true,
    baz: "hello"
};

"foo" in map == true;               // check object map for property name

'w' in "hello, world!" == true;     // check string for character

"wor" in "hello, world" == true;    // check string for sub-string

42 in -100..100 == true;            // check range for number
```
