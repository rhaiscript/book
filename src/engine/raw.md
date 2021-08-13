Raw `Engine`
===========

{{#include ../links.md}}

`Engine::new` creates a scripting [`Engine`] with common functionalities (e.g. printing to `stdout`
via `print` or `debug`).

In many controlled embedded environments, however, these may not be needed and unnecessarily occupy
application code storage space.

Use `Engine::new_raw` to create a _raw_ `Engine`, in which only a minimal set of
basic arithmetic and logical operators are supported (see below).

To add more functionalities to a _raw_ `Engine`, load [packages] into it.


Built-in Operators
------------------

Even with a raw [`Engine`], some operators are built in and are always available.

See [_Built-in Operators_][built-in operators] for a full list.


Differences with `Engine::new`
-----------------------------

`Engine::new` is equivalent to:

```rust no_run
use rhai::module_resolvers::FileModuleResolver;
use rhai::packages::StandardPackage;

// Create a raw scripting Engine
let mut engine = Self::new_raw();

// Use the file-based module resolver
engine.set_module_resolver(FileModuleResolver::new());

// Default print/debug implementations
engine.on_print(|text| println!("{}", text));

engine.on_debug(|text, source, pos| {
    if let Some(source) = source {
        println!("{}{:?} | {}", source, pos, text);
    } else if pos.is_none() {
        println!("{}", text);
    } else {
        println!("{:?} | {}", pos, text);
    }
});

// Register the Standard Package
let package = StandardPackage::new().as_shared_module();

engine.register_global_module(package);
```
