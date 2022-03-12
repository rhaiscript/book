Operator Overloading
===================

{{#include ../links.md}}

In Rhai, a lot of functionalities are actually implemented as functions, including basic operations
such as arithmetic calculations.

For example, in the expression "`a + b`", the `+` [operator] actually calls a function named "`+`"!

```rust,no_run
let x = a + b;

let x = +(a, b);        // <- the above is equivalent to this function call
```

Similarly, comparison [operators] including `==`, `!=` etc. are all implemented as functions,
with the stark exception of `&&` and `||`.

~~~admonish warning.small "`&&` and `||` cannot be overloaded"

Because they [_short-circuit_]({{rootUrl}}/language/logic.md#boolean-operators), `&&` and `||` are
handled specially and _not_ via a function.

Overriding them has no effect at all.
~~~


Overload Operator via Rust Function
----------------------------------

[Operator] functions cannot be defined in script because [operators] are usually not valid function names.

However, [operator] functions _can_ be registered via `Engine::register_fn`.

When a custom [operator] function is registered with the same name as an [operator],
it _overrides_ the built-in version.

```rust,no_run
use rhai::{Engine, EvalAltResult};

let mut engine = Engine::new();

fn strange_add(a: i64, b: i64) -> i64 { (a + b) * 42 }

engine.register_fn("+", strange_add);               // overload '+' operator for two integers!

let result: i64 = engine.eval("1 + 0");             // the overloading version is used

result == 42;

let result: f64 = engine.eval("1.0 + 0.0");         // '+' operator for two floats not overloaded

result == 1.0;

fn mixed_add(a: i64, b: bool) -> f64 { a + if b { 42 } else { 99 } }

engine.register_fn("+", mixed_add);                 // register '+' operator for an integer and a bool

let result: i64 = engine.eval("1 + true");          // <- normally an error...

result == 43;                                       //    ... but not now
```


```admonish danger.small "Considerations"

Use [operator] overloading for [custom types] only.

Be **very careful** when overriding built-in [operators] because users expect standard [operators] to
behave in a consistent and predictable manner, and will be annoyed if an expression involving `+`
turns into a subtraction, for example.  You may think it is amusing, but users who need to get
things done won't.

[Operator] overloading also impacts script optimization when using [`OptimizationLevel::Full`].
See the section on [script optimization] for more details.
```
