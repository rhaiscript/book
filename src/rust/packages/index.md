Packages
========

{{#include ../../links.md}}

The built-in library of Rhai is provided as various _packages_ that can be turned into _shared_
[modules], which in turn can be registered into the _global namespace_ of an [`Engine`] via
`Engine::register_global_module`.

Packages reside under `rhai::packages::*` and the trait `rhai::packages::Package` must be loaded in
order for packages to be used.

```admonish question.small "Rhai internals: Packages _are_ modules!"

Internally, a _package_ is a [module], with some conveniences to make it easier to define and use as
a standard _library_ for an [`Engine`].

Packages typically contain Rust functions that are callable within a Rhai script.
All _top-level_ functions in a package are available under the _global namespace_
(i.e. they're available without namespace qualifiers).

Sub-[modules] and [variables] are ignored in packages.
```


Share a Package Among Multiple `Engine`'s
----------------------------------------

`Engine::register_global_module` and `Engine::register_static_module` both require _shared_ [modules].

Once a package is created (e.g. via `Package::new`), it can create _shared_ [modules]
(via `Package::as_shared_module`) and register into multiple instances of [`Engine`],
even across threads (under the [`sync`] feature).

```admonish tip.small "Tip: Sharing package"

A package only has to be created _once_ and essentially shared among multiple [`Engine`] instances.

This is particularly useful when spawning large number of [raw `Engine`'s][raw `Engine`].
```

```rust,no_run
use rhai::Engine;
use rhai::packages::Package         // load the 'Package' trait to use packages
use rhai::packages::CorePackage;    // the 'core' package contains basic functionalities (e.g. arithmetic)

// Create a package - can be shared among multiple 'Engine' instances
let package = CorePackage::new();

let mut engines_collection: Vec<Engine> = Vec::new();

// Create 100 'raw' Engines
for _ in 0..100 {
    let mut engine = Engine::new_raw();

    // Register the package into the global namespace.
    // 'Package::as_shared_module' converts the package into a shared module.
    engine.register_global_module(package.as_shared_module());

    engines_collection.push(engine);
}
```
