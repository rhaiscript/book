Maximum Call Stack Depth
=======================

{{#include ../links.md}}


In Rhai, it is trivial for a function call to perform _infinite recursion_ such that all stack space
is exhausted.

```rust,no_run
// This is a function that, when called, recurse forever.
fn recurse_forever() {
    recurse_forever();
}
```


Limit How Stack Usage by Scripts
-------------------------------

Rhai by default limits function calls to a maximum depth of 64 levels (8 levels in debug build).

This limit may be changed via the `Engine::set_max_call_levels` method.

A script exceeding the maximum call stack depth will terminate with an error result.

This check can be disabled via the [`unchecked`] feature for higher performance (but higher risks as well).

```rust,no_run
let mut engine = Engine::new();

engine.set_max_call_levels(10);     // allow only up to 10 levels of function calls

engine.set_max_call_levels(0);      // allow no function calls at all (max depth = zero)
```


Setting Maximum Stack Depth
--------------------------

When setting this limit, care must be also taken to the evaluation depth of each _statement_
within a function. It is entirely possible for a malicious script to embed a recursive call deep
inside a nested expression or statement block (see [maximum statement depth]).
