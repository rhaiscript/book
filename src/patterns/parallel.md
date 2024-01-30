One `Engine` Instance Per Call
==============================

{{#include ../links.md}}


```admonish info "Usage scenario"

* A system where scripts are called a _lot_, in tight loops or in parallel.

* Keeping a global [`Engine`] instance is sub-optimal due to contention and locking.

* Scripts need to be executed independently from each other, perhaps concurrently.

* Scripts are used to [create Rust closures][`Func`] that are stored and may be called at any time,
  perhaps concurrently. In this case, the [`Engine`] instance is usually moved into the closure itself.
```

```admonish abstract "Key concepts"

* Rhai's [`AST`] structure is sharable &ndash; meaning that one copy of the [`AST`] can be run on
  multiple instances of [`Engine`] simultaneously.

* Rhai's [packages] and [modules] are also sharable.

* This means that [`Engine`] instances can be _decoupled_ from the base system ([packages] and
  [modules]) as well as the scripts ([`AST`]) so they can be created very cheaply.
```


Procedure
---------

* Gather up all common custom functions into a [custom package].

  * This [custom package] should also include standard [packages] needed. For example, to duplicate
    `Engine::new`, use a [`StandardPackage`]({{rootUrl}}/rust/packages/builtin.md).
  
  * [Packages] are sharable, so using a [custom package] is _much cheaper_ than registering all the
    functions one by one.

* Store a global [`AST`] for use with all [`Engine`] instances.

* Always use `Engine::new_raw` to create a [raw `Engine`], instead of `Engine::new` which is _much_
  more expensive. A [raw `Engine`] is _extremely_ cheap to create.

* Register the [custom package] with the [raw `Engine`] via `Package::register_into_engine`.


Examples
--------

```rust
use rhai::def_package;
use rhai::packages::{Package, StandardPackage};
use rhai::FuncRegistration;

// Define the custom package 'MyCustomPackage'.
def_package! {
    /// My own personal super-duper custom package
    // Aggregate other base packages simply by listing them after the colon.
    pub MyCustomPackage(module) : StandardPackage {
      // Register additional Rust functions.
      FuncRegistration::new("foo")
          .with_params(&["s: ImmutableString", "i64"])
          .set_into_module(module, |s: ImmutableString| foo(s.into_owned()));
    }
}

let ast = /* ... some AST ... */;

let my_custom_package = MyCustomPackage::new();

// The following loop creates 10,000 Engine instances!

for x in 0..10_000 {
    // Create a raw Engine - extremely cheap
    let mut engine = Engine::new_raw();

    // Register custom package - cheap
    my_custom_package.register_into_engine(&mut engine);

    // Evaluate script
    engine.run_ast(&ast)?;
}
```
