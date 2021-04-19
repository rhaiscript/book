Create a Module from Rust
========================

{{#include ../../links.md}}


The Easy Way &ndash; Create via Plugin
--------------------------------------

By far the simplest way to create a [module] is via a [plugin module]
which converts a normal Rust module into a Rhai [module] via procedural macros.


The Hard Way &ndash; Create via `Module` API
-------------------------------------------

Manually creating a [module] is possible via the `Module` public API, which is volatile and may
change from time to time.

For the complete `Module` public API, refer to the
[documentation](https://docs.rs/rhai/{{version}}/rhai/struct.Module.html) online.
