Lazy-Loading
============

{{#include ../links.md}}


Functions in [plugin modules] can now be resolved lazily: a [`Module`] can hold static _manifests_
(generated via `#[export_module(manifest)]`) alongside its normal functions.

Functions in _manifests_ are not registered but looked up only when called.


Reduce Runtime Memory Usage
---------------------------

Lazy-loading functions via a _manifest_, especially in large [plugin modules], can **significantly**
reduce runtime memory usage, to the order of 10-20x less.  This is especially critical on
memory-constrained targets.

Use the [`lazy_bench`]({{repoHome}}/examples/lazy_bench.rs) example to benchmark runtime
memory usage and performance of lazy-loaded functions.


Usage
-----

```admonish warning.side "Opt-in feature"
Lazy-loading is opt-in. The default behavior of [plugin modules] is to register all functions eagerly.
```

Every [package] defined via `def_package!` now has a `new_lazy` constructor that keeps the
functions of [plugin modules] within their _manifests_, looking them up only when called.

For example, [`Engine::new_raw`][raw `Engine`] with `StandardPackage::new_lazy` uses about 16x less
heap than `Engine::new`, and builds the standard library about 8x faster, at the cost of slower
function resolution the first time one is called.

Lazy-loading is supported by both the [`AST`] interpreter and [Rhai Grain].


Module Lazy Loading API
-----------------------

|                                    |             Normal              |                           Lazy-loaded                           |
| ---------------------------------- | :-----------------------------: | :-------------------------------------------------------------: |
| Create new [module]                |       `exported_module!`        |         `Module::from_manifest` + `exported_manifest!`          |
| Add functions to existing [module] | `combine_with_exported_module!` | `Module::combine_manifest` or `combine_with_exported_manifest!` |

`Module::register_lazy_functions` registers all lazy-loaded functions up-front, making them no
different from normal module functions.
