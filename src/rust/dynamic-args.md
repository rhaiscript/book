`Dynamic` Parameters in Rust Functions
=====================================

{{#include ../links.md}}

It is possible for Rust functions to contain parameters of type [`Dynamic`].
Any clonable value can be set into a [`Dynamic`] value.

Any parameter in a registered Rust function with a specific type has higher precedence over the
[`Dynamic`] type, so it is important to understand which _version_ of a function will be used.

For example, the `push` method of an [array] is implemented this way, which makes the function
applicable for all item types:

```rust,no_run
fn push(array: &mut Array, item: Dynamic) {
    array.push(item);
}
```


Examples
--------

A [`Dynamic`] value has less precedence than a value of a specific type, and parameter matching starts
from the left to the right. Candidate functions will be matched in order of parameter types.

Therefore, always leave [`Dynamic`] parameters as far to the right as possible.

```rust,no_run
use rhai::{Engine, RegisterFn, Dynamic};

// Different versions of the same function 'foo'
// will be matched in the following order.

fn foo1(x: i64, y: &str, z: bool) { }

fn foo2(x: i64, y: &str, z: Dynamic) { }

fn foo3(x: i64, y: Dynamic, z: bool) { }

fn foo4(x: i64, y: Dynamic, z: Dynamic) { }

fn foo1(x: Dynamic, y: &str, z: bool) { }

fn foo2(x: Dynamic, y: &str, z: Dynamic) { }

fn foo3(x: Dynamic, y: Dynamic, z: bool) { }

fn foo4(x: Dynamic, y: Dynamic, z: Dynamic) { }

let mut engine = Engine::new();

// Register all functions under the same name
// (the order does not matter)
engine
    .register_fn("foo", foo8)
    .register_fn("foo", foo7)
    .register_fn("foo", foo6)
    .register_fn("foo", foo5)
    .register_fn("foo", foo4)
    .register_fn("foo", foo3)
    .register_fn("foo", foo2)
    .register_fn("foo", foo1);
```
