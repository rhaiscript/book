Maximum Number of Functions
===========================

{{#include ../links.md}}

Rhai by default does not limit how many [functions] can be defined in a script.

This can be changed via the [`Engine::set_max_functions`][options] method. Notice that setting the
maximum number of [functions] to zero does _not_ indicate unlimited [functions], but disallows
defining any scripted [function] altogether.

A script attempting to load more than the maximum number of [functions] will terminate with a parse error.

This check can be disabled via the [`unchecked`] feature for higher performance (but higher risks as well).

```rust
let mut engine = Engine::new();

engine.set_max_functions(5);        // allow defining only up to 5 functions

engine.set_max_functions(0);        // disallow defining function (maximum = zero)

engine.set_max_functions(1000);     // set to a large number for effectively unlimited functions
```
