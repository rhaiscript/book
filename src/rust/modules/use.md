Make a Module Available to Scripts
==================================

{{#include ../../links.md}}


Use Case 1 &ndash; Make It Globally Available
--------------------------------------------

`Engine::register_global_module` registers a shared [module] into the
[_global_ namespace][function namespace].

This is by far the easiest way to expose a [module]'s functionalities to Rhai.

```admonish tip.small "Tip: No qualifiers"

All [functions], [variables]/[constants] and [type iterators] can be accessed without
_namespace qualifiers_.
```

```admonish warning.small

[Sub-modules][module] are **ignored**.
```

```rust
use rhai::{Engine, Module};

let mut module = Module::new();             // new module

// Use 'Module::set_native_fn' to add functions.
let hash = module.set_native_fn("inc", |x: i64| Ok(x + 1));

// Remember to update the parameter names/types and return type metadata
// when using the 'metadata' feature.
// 'Module::set_native_fn' by default does not set function metadata.
module.update_fn_metadata(hash, &["x: i64", "i64"]);

// Use 'Module::set_var' to add variables.
module.set_var("MYSTIC_NUMBER", 41_i64);

// Register the module into the global namespace of the Engine.
let mut engine = Engine::new();
engine.register_global_module(module.into());

// No need to import module...
engine.eval::<i64>("inc(MYSTIC_NUMBER)")? == 42;
```

### Equivalent to `Engine::register_XXX`

```admonish question.side.wide "Trivia"

`Engine::register_fn` etc. are actually implemented by adding functions to an
internal [module]!
```

Registering a [module] via `Engine::register_global_module` is essentially the _same_
as calling `Engine::register_fn` (or any of the `Engine::register_XXX` API) individually
on each top-level function within that [module].

```rust
// The above is essentially the same as:
let mut engine = Engine::new();

engine.register_fn("inc", |x: i64| x + 1);

engine.eval::<i64>("inc(41)")? == 42;       // no need to import module
```


Use Case 2 &ndash; Make It a Static Namespace
--------------------------------------------

`Engine::register_static_module` registers a [module] and under a specific
[module namespace][function namespace].

```rust
use rhai::{Engine, Module};

let mut module = Module::new();             // new module

// Use 'Module::set_native_fn' to add functions.
let hash = module.set_native_fn("inc", |x: i64| Ok(x + 1));

// Remember to update the parameter names/types and return type metadata
// when using the 'metadata' feature.
// 'Module::set_native_fn' by default does not set function metadata.
module.update_fn_metadata(hash, &["x: i64", "i64"]);

// Use 'Module::set_var' to add variables.
module.set_var("MYSTIC_NUMBER", 41_i64);

// Register the module into the Engine as the static module namespace path
// 'services::calc'
let mut engine = Engine::new();
engine.register_static_module("services::calc", module.into());

// Refer to the 'services::calc' module...
engine.eval::<i64>("services::calc::inc(services::calc::MYSTIC_NUMBER)")? == 42;
```

### Expose functions to the global namespace

```admonish tip.side.wide "Tip: Type iterators"

[Type iterators] are special &mdash; they are _always_ exposed to the
[_global_ namespace][function namespace].
```

The [`Module`] API can optionally expose functions to the [_global_ namespace][function namespace]
by setting the `namespace` parameter to `FnNamespace::Global`.

This way, [getters/setters] and [indexers] for [custom types] can work as expected.

```rust
use rhai::{Engine, Module, FnNamespace};

let mut module = Module::new();             // new module

// Use 'Module::set_native_fn' to add functions.
let hash = module.set_native_fn("inc", |x: &mut i64| Ok(x + 1));

// Remember to update the parameter names/types and return type metadata
// when using the 'metadata' feature.
// 'Module::set_native_fn' by default does not set function metadata.
module.update_fn_metadata(hash, &["x: &mut i64", "i64"]);

// Expose method 'inc' to the global namespace (default is 'FnNamespace::Internal')
module.update_fn_namespace(hash, FnNamespace::Global);

// Use 'Module::set_var' to add variables.
module.set_var("MYSTIC_NUMBER", 41_i64);

// Register the module into the Engine as a static module namespace 'calc'
let mut engine = Engine::new();
engine.register_static_module("calc", module.into());

// 'inc' works when qualified by the namespace
engine.eval::<i64>("calc::inc(calc::MYSTIC_NUMBER)")? == 42;

// 'inc' also works without a namespace qualifier
// because it is exposed to the global namespace
engine.eval::<i64>("let x = calc::MYSTIC_NUMBER; x.inc()")? == 42;
engine.eval::<i64>("let x = calc::MYSTIC_NUMBER; inc(x)")? == 42;
```


Use Case 3 &ndash; Make It Dynamically Loadable
----------------------------------------------

In order to dynamically load a custom module, there must be a [module resolver] which serves
the module when loaded via `import` statements.

The easiest way is to use, for example, the [`StaticModuleResolver`][module resolver] to hold such
a custom module.

```rust
use rhai::{Engine, Scope, Module};
use rhai::module_resolvers::StaticModuleResolver;

let mut module = Module::new();             // new module
module.set_var("answer", 41_i64);           // variable 'answer' under module
module.set_native_fn("inc", |x: i64| {      // use 'Module::set_native_fn' to add functions
    Ok(x + 1)
});

// Create the module resolver
let mut resolver = StaticModuleResolver::new();

// Add the module into the module resolver under the name 'question'
// They module can then be accessed via: 'import "question" as q;'
resolver.insert("question", module);

// Set the module resolver into the 'Engine'
let mut engine = Engine::new();
engine.set_module_resolver(resolver);

// Use namespace-qualified variables
engine.eval::<i64>(
r#"
    import "question" as q;
    q::answer + 1
"#)? == 42;

// Call namespace-qualified functions
engine.eval::<i64>(
r#"
    import "question" as q;
    q::inc(q::answer)
"#)? == 42;
```
