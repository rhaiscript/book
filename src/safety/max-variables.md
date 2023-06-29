Maximum Number of Variables
===========================

{{#include ../links.md}}

Rhai by default does not limit how many [variables]/[constants] can be defined within a single [`Scope`].

This can be changed via the [`Engine::set_max_variables`][options] method. Notice that setting the
maximum number of [variables] to zero does _not_ indicate unlimited [variables], but disallows
defining any [variable] altogether.

A script attempting to define more than the maximum number of [variables]/[constants] will terminate
with an error result.

This check can be disabled via the [`unchecked`] feature for higher performance (but higher risks as well).

```rust
let mut engine = Engine::new();

engine.set_max_variables(5);      // allow defining only up to 5 variables

engine.set_max_variables(0);      // disallow defining any variable (maximum = zero)

engine.set_max_variables(1000);   // set to a large number for effectively unlimited variables
```

```admonish warning.small "Function calls are separate scopes"

Each [function] call creates a new, empty [`Scope`].

Therefore, [variables]/[constants] defined within [functions] are counted afresh.

Care must be taken to avoid deeply-nested (or recursive) [function] calls from creating too many
[variables]/[constants] while staying within the limit of each individual [`Scope`].
```

```admonish warning.small "Function call arguments count as variables"

The parameters of a [function] also count as [variables] within the [function]'s [`Scope`].

Thus the maximum number of [variables]/[constants] allowed is reduced by the number of parameters of the [function].
```

~~~admonish tip.small "Tip: Reusing a variable doesn't count"

It is possible to _reuse_ a [variable] such that it is counted only once.

```rust
let x = 42;     // counted as 1 variable
let y = 123;

let x = 0;      // previous 'x' reused: not counted as new variable
```
~~~
