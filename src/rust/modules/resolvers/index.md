Module Resolvers
================

{{#include ../../../links.md}}

~~~admonish info.side "`import`"

See the section on [_Importing Modules_][`import`] for more details.
~~~

When encountering an [`import`] statement, Rhai attempts to _resolve_ the [module] based on the path string.

_Module Resolvers_ are service types that implement the [`ModuleResolver`][traits] trait.


Set into `Engine`
-----------------

An [`Engine`]'s module resolver is set via a call to `Engine::set_module_resolver`:

```rust
use rhai::module_resolvers::{DummyModuleResolver, StaticModuleResolver};

// Create a module resolver
let resolver = StaticModuleResolver::new();

// Register functions into 'resolver'...

// Use the module resolver
engine.set_module_resolver(resolver);

// Effectively disable 'import' statements by setting module resolver to
// the 'DummyModuleResolver' which acts as... well... a dummy.
engine.set_module_resolver(DummyModuleResolver::new());
```
