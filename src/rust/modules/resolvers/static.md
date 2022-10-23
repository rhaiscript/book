`StaticeModuleResolver`
=======================

{{#include ../../../links.md}}


~~~admonish abstract.small "Useful for `no-std`"

`StaticModuleResolver` is often used with [`no_std`] in embedded environments
without a file system.
~~~

Loads [modules] that are statically added.

Functions are searched in the [_global_ namespace][function namespace] by default.

```rust
use rhai::{Module, module_resolvers::StaticModuleResolver};

let module: Module = create_a_module();

let mut resolver = StaticModuleResolver::new();
resolver.insert("my_module", module);

engine.set_module_resolver(resolver);
```
