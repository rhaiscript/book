Maximum Number of Modules
========================

{{#include ../links.md}}

Rhai by default does not limit how many [modules] can be loaded via [`import`] statements.

This can be changed via the [`Engine::set_max_modules`][options] method. Notice that setting the
maximum number of modules to zero does _not_ indicate unlimited modules, but disallows loading any
module altogether.

A script attempting to load more than the maximum number of modules will terminate with an error result.

This limit can also be used to stop [`import`-loops][`import`] (i.e. cycles of modules referring to
each other).

This check can be disabled via the [`unchecked`] feature for higher performance (but higher risks as well).

```rust
let mut engine = Engine::new();

engine.set_max_modules(5);      // allow loading only up to 5 modules

engine.set_max_modules(0);      // disallow loading any module (maximum = zero)

engine.set_max_modules(1000);   // set to a large number for effectively unlimited modules
```
