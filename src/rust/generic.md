Register a Generic Rust Function
================================

{{#include ../links.md}}

```admonish info.side.wide "No monomorphization"

Due to its dynamic nature, Rhai cannot monomorphize generic functions automatically.

Monomorphization of generic functions must be performed manually.
```

Rust generic functions can be used in Rhai, but separate instances for each concrete type must be
registered separately.

This essentially _overloads_ the function with different parameter types as Rhai does not natively
support generics but Rhai does support _[function overloading]_.

The example below shows how to register multiple functions (or, in this case, multiple overloaded
versions of the same function) under the same name.

```rust
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
