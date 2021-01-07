One Engine Instance Per Call
===========================

{{#include ../links.md}}


Usage Scenario
--------------

* A system where scripts are called a _lot_, in tight loops or in parallel.

* Keeping a global [`Engine`] instance is sub-optimal due to contention and locking.

* Scripts need to be executed independently from each other, perhaps concurrently.

* Scripts are used to [create Rust closure][`Func`] that are stored and may be called at any time, perhaps concurrently.
  In this case, the [`Engine`] instance is usually moved into the closure itself.


Key Concepts
------------

* Create a single instance of each standard [package] required.
  To duplicate `Engine::new`, create a [`StandardPackage`]({{rootUrl}}/rust/packages/builtin.md).

* Gather up all common custom functions into a [custom package].

* Store a global `AST` for use with all engines.

* Always use `Engine::new_raw` to create a [raw `Engine`], instead of `Engine::new` which is _much_ more expensive.
  A [raw `Engine`] is _extremely_ cheap to create.
  
  Registering the [`StandardPackage`]({{rootUrl}}/rust/packages/builtin.md) into a [raw `Engine`] via
  `Engine::register_global_module` is essentially the same as `Engine::new`.
  
  However, because packages are shared, using existing package is _much cheaper_ than
  registering all the functions one by one.

* Register the required packages with the [raw `Engine`] via `Engine::register_global_module`,
  using `Package::as_shared_module` to obtain a shared [module].


Examples
--------

```rust
use rhai::packages::{Package, StandardPackage};

let ast = /* ... some AST ... */;
let std_pkg = StandardPackage::new();
let custom_pkg = MyCustomPackage::new();

let make_call = |x: i64| -> Result<(), Box<EvalAltResult>> {
    // Create a raw Engine - extremely cheap
    let mut engine = Engine::new_raw();

    // Register packages as global modules - cheap
    engine.register_global_module(std_pkg.as_shared_module());
    engine.register_global_module(custom_pkg.as_shared_module());

    // Create custom scope - cheap
    let mut scope = Scope::new();

    // Push variable into scope - relatively cheap
    scope.push("x", x);

    // Evaluate script.
    engine.consume_ast_with_scope(&mut scope, &ast)
};

// The following loop creates 10,000 Engine instances!
for x in 0..10_000 {
    make_call(x)?;
}
```
