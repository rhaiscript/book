Logic Operators
==============

{{#include ../links.md}}

Comparison Operators
--------------------

| Operator | Description               |
| :------: | ------------------------- |
|   `==`   | equals to                 |
|   `!=`   | not equals to             |
|   `>`    | greater than              |
|   `>=`   | greater than or equals to |
|   `<`    | less than                 |
|   `<=`   | less than or equals to    |

Comparison between most values of the same type work out-of-the-box for all [standard types]
supported by the system.

### Floating-point numbers can inter-operate with integers

Comparing a floating-point number (`FLOAT` or [`Decimal`][rust_decimal]) with an integer is also supported.

```rust,no_run
42 == 42.0;         // true

42.0 == 42;         // true

42.0 > 42;          // false

42 >= 42.0;         // true

42.0 < 42;          // false
```

### Different types cannot be compared (always `false`)

Comparing two values of _different_ data types, or of unknown data types, always results in `false`,
except for `!=` (not equals) which results in `true`. This is in line with intuition.

```rust,no_run
42 > "42";          // false: i64 cannot be compared with string

42 <= "42";         // false: i64 cannot be compared with string

let ts = new_ts();  // custom type

ts == 42;           // false: different types cannot be compared

ts != 42;           // true: different types cannot be compared

ts == ts;           // false: unless '==' is defined for the custom type
```

### Caution: Beware operators for custom types

The default comparison behavior (i.e. returning `false` instead of raising an error)
may be counter-intuitive for [custom types] because all comparisons default to `false`
(`true` for `!=`), unless the corresponding operators are defined for the type.

```rust,no_run
let ts = new_ts();  // custom type that doesn't have any operators defined

ts == ts;           // false: unless '==' is defined for the custom type

ts != ts;           // true: unless '!=' is defined for the custom type
```

Operators are completely separate from each other.  For example:

* `!=` does NOT equal to `!(==)`
* `>` does NOT equal to `!(<=)`
* `<=` does NOT equal to `<` plus `==`

Therefore, if a [custom type] misses an operator definition, the default result is simply returned.

This behavior can be counter-intuitive.

```rust,no_run
let ts = new_ts();  // custom type with '<=' and '==' defined

ts <= ts;           // true: '<=' defined

ts < ts;            // false: '<' not defined, even though '<=' is true

ts == ts;           // true: '==' defined

ts != ts;           // true: '!=' not defined, even though '==' is true
```

It is strongly recommended that, when defining operators for [custom types], always define the full set
of six operators (or at least the `==` and `!=` pair) together.

### Caution: Built-in operators behave differently when using a raw `Engine`

When using a [raw `Engine`] without loading any [packages], comparisons can only be made between a limited
set of types (see [built-in operators]), and both operands must be of the same type (including
floating-point numbers).  Otherwise the result is always `false` (`true` for `!=`).

Therefore, this is an area where a standard [`Engine`] behaves differently from a [raw `Engine`]
transparently (i.e. without causing errors).

```rust,no_run
// The following under Engine::new_raw()

42.0 == 42.0;       // true: built in

42.0 == 42;         // false: comparison between FLOAT and INT not built in
                    // in a standard Engine, this is true

42 < 123;           // true: built in

42 < 123.0;         // false: comparison between INT and FLOAT not built in
                    // in a standard Engine, this is true

"hello" > "foo";    // true: built in
```

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
