Custom Operators
================

{{#include ../links.md}}

For use as a DSL (Domain-Specific Languages), it is sometimes more convenient to augment Rhai with
customized operators performing specific logic.

`Engine::register_custom_operator` registers a keyword as a custom operator, giving it a particular
_precedence_ (which cannot be zero).


Example
-------

```rust no_run
use rhai::Engine;

let mut engine = Engine::new();

// Register a custom operator '#' and give it a precedence of 160
// (i.e. between +|- and *|/)
// Also register the implementation of the customer operator as a function
engine.register_custom_operator("#", 160)?
      .register_fn("#", |x: i64, y: i64| (x * y) - (x + y));

// The custom operator can be used in expressions
let result = engine.eval_expression::<i64>("1 + 2 * 3 # 4 - 5 / 6")?;
//                                                    ^ custom operator

// The above is equivalent to: 1 + ((2 * 3) # 4) - (5 / 6)
result == 15;
```


Alternatives to a Custom Operator
--------------------------------

Custom operators are merely _syntactic sugar_.  They map directly to registered functions.

```rust no_run
┌──────┐
│ Rust │
└──────┘

engine.register_custom_operator("foo", 160)?;

┌──────┐
│ Rhai │
└──────┘

1 + 2 * 3 foo 4 - 5 / 6     // use custom operator

1 + foo(2 * 3, 4) - 5 / 6   // <- above is equivalent to this function call
```

A script using custom operators can always be pre-processed, via a pre-processor application,
into a syntax that uses the corresponding function calls.

Using `Engine::register_custom_operator` merely enables a convenient shortcut.


Must be a Valid Identifier or Reserved Symbol
--------------------------------------------

All custom operators must be _identifiers_ that follow the same naming rules as [variables].

Alternatively, they can also be [reserved symbols]({{rootUrl}}/appendix/operators.md#symbols),
[disabled operators or keywords][disable keywords and operators].

```rust no_run
engine.register_custom_operator("foo", 20)?;    // 'foo' is a valid custom operator

engine.register_custom_operator("#", 20)?;      // the reserved symbol '#' is also
                                                // a valid custom operator

engine.register_custom_operator("+", 30)?;      // <- error: '+' is an active operator

engine.register_custom_operator("=>", 30)?;     // <- error: '=>' is an active symbol
```


Binary Operators Only
---------------------

All custom operators must be _binary_ (i.e. they take two operands).
_Unary_ custom operators are not supported.

```rust no_run
engine.register_custom_operator("#", 160)?
      .register_fn("#", |x: i64| x * x);

engine.eval::<i64>("1 + 2 * 3 # 4 - 5 / 6")?;   // error: function '# (i64, i64)' not found
```


Operator Precedence
-------------------

All operators in Rhai has a _precedence_ indicating how tightly they bind.

A higher precedence binds more tightly than a lower precedence, so `*` and `/` binds before `+` and `-` etc.

When registering a custom operator, the operator's precedence must also be provided.

The following _precedence table_ shows the built-in precedence of standard Rhai operators:

| Category            |                Operators                 | Binding | Precedence (0-255) |
| ------------------- | :--------------------------------------: | :-----: | :----------------: |
| Logic and bit masks | <code>\|\|</code>,  <code>\|</code>, `^` |  left   |         30         |
| Logic and bit masks |                `&&`, `&`                 |  left   |         60         |
| Comparisons         |                `==`, `!=`                |  left   |         90         |
| Containment         |                   `in`                   |  left   |        110         |
| Comparisons         |           `>`, `>=`, `<`, `<=`           |  left   |        130         |
| Ranges              |               `..`, `..=`                |  left   |        140         |
| Arithmetic          |                 `+`, `-`                 |  left   |        150         |
| Arithmetic          |              `*`, `/`, `%`               |  left   |        180         |
| Arithmetic          |                   `**`                   |  right  |        190         |
| Bit-shifts          |                `<<`, `>>`                |  left   |        210         |
| Unary operators     |           unary `+`, `-`, `!`            |  right  |      highest       |
| Object field access |                   `.`                    |  right  |      highest       |
