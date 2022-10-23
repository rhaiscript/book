`DylibModuleResolver`
=====================

{{#include ../../../links.md}}


~~~admonish warning.small "Requires external crate `rhai-dylib`"

`DylibModuleResolver` resides in the [`rhai-dylib`] crate which must be specified
as a dependency:

```toml
[dependencies]
rhai-dylib = { version = "0.1" }
```
~~~

```admonish danger.small "Linux or Windows only"

[`rhai-dylib`] currently supports only Linux and Windows.
```

Parallel to how the [`FileModuleResolver`](file.md) works, `DylibModuleResolver` loads external
native Rust [modules] from compiled _dynamic shared libraries_ (e.g. `.so` in Linux and `.dll` in
Windows).

Therefore, [`FileModuleResolver`](file.md`) loads Rhai script files while `DylibModuleResolver`
loads native Rust shared libraries.  It is very common to have the two work together.


Example
-------

```rust
use rhai::{Engine, Module};
use rhai::module_resolvers::{FileModuleResolver, ModuleResolversCollection};
use rhai_dylib::module_resolvers::DylibModuleResolver;

let mut engine = Engine::new();

// Use a module resolvers collection
let mut resolvers = ModuleResolversCollection::new();

// First search for script files in the file system
resolvers += FileModuleResolver::new();

// Then search for shared-library plugins in the file system
resolvers += DylibModuleResolver::new();

// Set the module resolver into the engine
engine.set_module_resolver(resolvers);


┌─────────────┐
│ Rhai Script │
└─────────────┘

// If there is 'path/to/my_module.rhai', load it.
// Otherwise, check for 'path/to/my_module.so' on Linux
// ('path/to/my_module.dll' on Windows).
import "path/to/my_module" as m;

m::greet();
```
