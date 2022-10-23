Create Dynamically Loadable Rhai Libraries
===========================================

{{#include ../links.md}}

```admonish danger.small "Linux or Windows only"

`rhai-dylib` currently supports only Linux and Windows.
```

`rhai-dylib` is an independent crate that demonstrates an API to register Rhai functionalities via
_dynamic shared libraries_ (i.e. `.so` in Linux or `.dll` in Windows).

In other words, functions and [modules] can be defined in external libraries that are loaded
dynamically at _runtime_, allowing for great flexibility at the cost of depending on the unstable
Rust ABI.

A [module resolver] is also included.

> On `crates.io`: [`rhai-dylib`](https://crates.io/crates/rhai-dylib)
>
> On `GitHub`: [`rhaiscript/rhai-dylib`](https://github.com/rhaiscript/rhai-dylib)
>
> API trait name: `rhai_dylib::Plugin`
