Sand-Boxing &ndash; Block Access to External Data
================================================

{{#include ../links.md}}

Rhai is _sand-boxed_ so a script can never read from outside its own environment.

Furthermore, an [`Engine`] created non-`mut` cannot mutate any state, including itself
(and therefore it is also _re-entrant_).

It is highly recommended that [`Engine`]'s be created immutable as much as possible.

```rust,no_run
let mut engine = Engine::new();

// Use the fluent API to configure an 'Engine'
engine.register_get("field", get_field)
      .register_set("field", set_field)
      .register_fn("do_work", action);

// Then turn it into an immutable instance
let engine = engine;

// 'engine' is immutable...
```


Using Rhai to Control External Environment
-----------------------------------------

How does a _sand-boxed_, immutable [`Engine`] control the external environment?
This is necessary in order to use Rhai as a _dynamic control layer_ over a Rust core system.

There are two general patterns, both involving wrapping the external system
in a shared, interior-mutated object (e.g. `Rc<RefCell<T>>`):

* [Control Layer]({{rootUrl}}/patterns/control.md) pattern.

* [Singleton Command Object]({{rootUrl}}/patterns/singleton.md) pattern.
