Function Overloading
====================

{{#include ../links.md}}


Functions registered with the [`Engine`] can be _overloaded_ as long as the _signature_ is unique,
i.e. different functions can have the same name as long as their parameters are of different numbers
(i.e. _arity_) or  different types.

New definitions _overwrite_ previous definitions of the same name, same arity and same parameter types.

~~~admonish tip "Tip: Overloading as a form of default parameter values"

Rhai does not support default values for function parameters.

However it is extremely easy to _simulate_ default parameter values via multiple overloaded
registrations of the same function name.

```rust
// The following definition of 'foo' is equivalent to the pseudo-code:
//   fn foo(x = 42_i64, y = "hello", z = true) -> i64 { ... }

fn foo3(x: i64, y: &str, z: bool) -> i64 { ... }
fn foo2(x: i64, y: &str) -> i64 { foo3(x, y, true) }
fn foo1(x: i64) -> i64 { foo2(x, "hello") }
fn foo0() -> i64 { foo1(42) }

engine.register_fn("foo", foo0)     // no parameters
      .register_fn("foo", foo1)     // 1 parameter
      .register_fn("foo", foo2)     // 2 parameters
      .register_fn("foo", foo3);    // 3 parameters
```
~~~
