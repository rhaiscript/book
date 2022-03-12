If Expression
=============

{{#include ../links.md}}

Like Rust, [`if`] statements can also be used as _expressions_, replacing the `? :` conditional
operators in other C-like languages.

~~~admonish tip.small "Tip: Disable `if` expressions"

[`if`] expressions can be disabled via [`Engine::set_allow_if_expression`][options].
~~~

```rust,no_run
// The following is equivalent to C: int x = 1 + (decision ? 42 : 123) / 2;
let x = 1 + if decision { 42 } else { 123 } / 2;
x == 22;

let x = if decision { 42 }; // no else branch defaults to '()'
x == ();
```
