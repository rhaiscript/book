Create Dynamically Loadable Rhai Libraries
===========================================

{{#include ../links.md}}

`rhai-dylib` is an independent crate that demonstrates an API to register Rhai functionalities via
_dynamic libraries_.

In other words, functions and [modules] can be defined in external libraries that are loaded
dynamically at _runtime_, allowing for great flexibility at the cost of depending on the unstable
Rust ABI.

> On `crates.io`: [`rhai-dylib`](https://crates.io/crates/rhai-dylib)
>
> On `GitHub`: [`rhaiscript/rhai-dylib`](https://github.com/rhaiscript/rhai-dylib)
>
> API trait name: `rhai_dylib::Plugin`
