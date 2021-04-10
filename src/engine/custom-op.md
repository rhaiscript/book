Custom Operators
================

{{#include ../links.md}}

For use as a DSL (Domain-Specific Languages), it is sometimes more convenient to augment Rhai with
customized operators performing specific logic.

`Engine::register_custom_operator` registers a keyword as a custom operator, giving it a particular
_precedence_ (which cannot be zero).


Example
-------

```rust , no_run
use rhai::Engine;

let mut engine = Engine::new();

// Register a custom operator named 'foo' and give it a precedence of 160
// (i.e. between +|- and *|/)
// Also register the implementation of the customer operator as a function
engine.register_custom_operator("foo", 160)?
      .register_fn("foo", |x: i64, y: i64| (x * y) - (x + y));

// The custom operator can be used in expressions
let result = engine.eval_expression::<i64>("1 + 2 * 3 foo 4 - 5 / 6")?;
//                                                    ^ custom operator

// The above is equivalent to: 1 + ((2 * 3) foo 4) - (5 / 6)
result == 15;
```


Alternatives to a Custom Operator
--------------------------------

Custom operators are merely _syntactic sugar_.  They map directly to registered functions.

Therefore, the following are equivalent (assuming `foo` has been registered as a custom operator):

```rust , no_run
1 + 2 * 3 foo 4 - 5 / 6     // use custom operator

1 + foo(2 * 3, 4) - 5 / 6   // use function call
```

A script using custom operators can always be pre-processed, via a pre-processor application,
into a syntax that uses the corresponding function calls.

Using `Engine::register_custom_operator` merely enables a convenient shortcut.


Must be a Valid Identifier or Reserved Symbol
--------------------------------------------

All custom operators must be _identifiers_ that follow the same naming rules as [variables].

Alternatively, they can also be [reserved symbols]({{rootUrl}}/appendix/operators.md#symbols),
[disabled operators or keywords][disable keywords and operators].

```rust , no_run
engine.register_custom_operator("foo", 20);     // 'foo' is a valid custom operator

engine.register_custom_operator("#", 20);       // the reserved symbol '#' is also
                                                // a valid custom operator

engine.register_custom_operator("+", 30);       // <- error: '+' is an active operator

engine.register_custom_operator("=>", 30);      // <- error: '=>' is an active symbol
```


Binary Operators Only
---------------------

All custom operators must be _binary_ (i.e. they take two operands).
_Unary_ custom operators are not supported.

```rust , no_run
engine.register_custom_operator("foo", 160)?
      .register_fn("foo", |x: i64| x * x);

engine.eval::<i64>("1 + 2 * 3 foo 4 - 5 / 6")?; // error: function 'foo (i64, i64)' not found
```


Operator Precedence
-------------------

All operators in Rhai has a _precedence_ indicating how tightly they bind.

A higher precedence binds more tightly than a lower precedence, so `*` and `/` binds before `+` and `-` etc.

When registering a custom operator, the operator's precedence must also be provided.

The following _precedence table_ shows the built-in precedence of standard Rhai operators:

| Category            |                                        Operators                                         | Precedence (0-255) |
| ------------------- | :--------------------------------------------------------------------------------------: | :----------------: |
| Assignments         | `=`, `+=`, `-=`, `*=`, `/=`, `**=`, `%=`,<br/>`<<=`, `>>=`, `&=`, <code>\|=</code>, `^=` |         0          |
| Logic and bit masks |                         <code>\|\|</code>,  <code>\|</code>, `^`                         |         30         |
| Logic and bit masks |                                        `&&`, `&`                                         |         60         |
| Comparisons         |                                        `==`, `!=`                                        |         90         |
| Containment         |                                           `in`                                           |        110         |
| Comparisons         |                                   `>`, `>=`, `<`, `<=`                                   |        130         |
| Arithmetic          |                                         `+`, `-`                                         |        150         |
| Arithmetic          |                                      `*`, `/`, `%`                                       |        180         |
| Arithmetic          |                                 `**` _(binds to right)_                                  |        190         |
| Bit-shifts          |                                        `<<`, `>>`                                        |        210         |
| Unary operators     |                          unary `+`, `-`, `!` _(binds to right)_                          |   second highest   |
| Object field access |                                  `.` _(binds to right)_                                  |      highest       |
