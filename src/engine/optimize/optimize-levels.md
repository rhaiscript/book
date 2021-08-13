Optimization Levels
==================

{{#include ../../links.md}}

There are three levels of optimization: `None`, `Simple` and `Full`.

* `None` is obvious &ndash; no optimization on the AST is performed.

* `Simple` (default) performs only relatively _safe_ optimizations without causing side-effects
  (i.e. it only relies on static analysis and [built-in operators] for [constant] [standard types],
  and will not perform any external function calls).

  However, it is important to bear the caveat in mind that, when _constants propagation_ is
  performed, and if the [constants] are modified later on (yes, it is possible, via Rust functions),
  the modified values will _not_ show up in the optimized script.  Only the initialization values
  of [constants] are ever retained.

  Furthermore, overriding a [built-in operator] in the [`Engine`] afterwards has no effect after the
  optimizer replaces an expression with its calculated value.

* `Full` is _much_ more aggressive, _including_ calling external functions on [constant] arguments
  to determine their results. One benefit to this is that many more optimization opportunities
  arise, especially with regards to comparison operators.


Set Optimization Level
---------------------

An [`Engine`]'s optimization level is set via a call to `Engine::set_optimization_level`:

```rust no_run
// Turn on aggressive optimizations
engine.set_optimization_level(rhai::OptimizationLevel::Full);
```
