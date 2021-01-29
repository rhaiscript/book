Logic Operators
==============

{{#include ../links.md}}

Comparison Operators
-------------------

| Operator | Description               |
| :------: | ------------------------- |
|   `==`   | equals to                 |
|   `!=`   | not equals to             |
|   `>`    | greater than              |
|   `>=`   | greater than or equals to |
|   `<`    | less than                 |
|   `<=`   | less than or equals to    |

Comparing most values of the same data type work out-of-the-box for all [standard types] supported by the system.

However, if using a [raw `Engine`] without loading any [packages], comparisons can only be made between a limited
set of types (see [built-in operators]).

```rust
42 == 42;               // true

42 > 42;                // false

"hello" > "foo";        // true

"42" == 42;             // false
```

Comparing two values of _different_ data types, or of unknown data types, always results in `false`,
except for '`!=`' (not equals) which results in `true`. This is in line with intuition.

```rust
42 == 42.0;             // false - i64 cannot be compared with f64

42 != 42.0;             // true - i64 cannot be compared with f64

42 > "42";              // false - i64 cannot be compared with string

42 <= "42";             // false - i64 cannot be compared with string

let ts = new_ts();      // custom type

ts == 42;               // false - types cannot be compared

ts != 42;               // true - types cannot be compared
```

Boolean operators
-----------------

|     Operator      | Description   | Arity  | Short-Circuits? |
| :---------------: | ------------- | :----: | :-------------: |
|   `!` (prefix)    | boolean _NOT_ | unary  |       no        |
|       `&&`        | boolean _AND_ | binary |       yes       |
|        `&`        | boolean _AND_ | binary |       no        |
| <code>\|\|</code> | boolean _OR_  | binary |       yes       |
|  <code>\|</code>  | boolean _OR_  | binary |       no        |

Double boolean operators `&&` and `||` _short-circuit_ &ndash; meaning that the second operand will not be evaluated
if the first one already proves the condition wrong.

Single boolean operators `&` and `|` always evaluate both operands.

```rust
a() || b();             // b() is not evaluated if a() is true

a() && b();             // b() is not evaluated if a() is false

a() | b();              // both a() and b() are evaluated

a() & b();              // both a() and b() are evaluated
```

All boolean operators are [built in][built-in operators] for the `bool` data type.
