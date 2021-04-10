Register a Generic Rust Function
===============================

{{#include ../links.md}}

Rust generic functions can be used in Rhai, but separate instances for each concrete type must be registered separately.

This essentially _overloads_ the function with different parameter types as Rhai does not natively support generics
but Rhai does support _function overloading_.

```rust , no_run
use std::fmt::Display;

use rhai::Engine;

fn show_it<T: Display>(x: &mut T) {
    println!("put up a good show: {}!", x)
}

let mut engine = Engine::new();

engine.register_fn("print", show_it::<i64>)
      .register_fn("print", show_it::<bool>)
      .register_fn("print", show_it::<ImmutableString>);
```

The above example shows how to register multiple functions
(or, in this case, multiple overloaded versions of the same function)
under the same name.
