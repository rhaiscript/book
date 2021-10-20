Plugins
=======

{{#include ../links.md}}

Rhai contains a robust _plugin_ system that greatly simplifies registration of custom
functionality.

Instead of using the large `Engine::register_XXX` API or the parallel [`Module`] API,
a _plugin_ simplifies the work of creating and registering new functionality in an [`Engine`].

Plugins are processed via a set of procedural macros under the `rhai::plugin` module. These
allow registering Rust functions directly in the Engine, or adding Rust modules as packages.

There are two types of plugins:

1) [Plugin Functions]

2) [Plugin Modules]
