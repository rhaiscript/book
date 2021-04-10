Turn Off Script Optimizations
============================

{{#include ../../links.md}}

When scripts:

* are known to be run only _once_,

* are known to contain no dead code,

* do not use constants in calculations

the optimization pass may be a waste of time and resources.  In that case, turn optimization off
by setting the optimization level to [`OptimizationLevel::None`].

Alternatively, turn off optimizations via the [`no_optimize`] feature.

```rust , no_run
let engine = rhai::Engine::new();

// Turn off the optimizer
engine.set_optimization_level(rhai::OptimizationLevel::None);
```
