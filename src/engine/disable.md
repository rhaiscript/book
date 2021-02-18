Disable Certain Keywords and/or Operators
========================================

{{#include ../links.md}}

For certain embedded usage, it is sometimes necessary to restrict the language to a strict subset of Rhai
to prevent usage of certain language features.

Rhai supports surgically disabling a keyword or operator via the `Engine::disable_symbol` method.

```rust,no_run
use rhai::Engine;

let mut engine = Engine::new();

engine
    .disable_symbol("if")       // disable the 'if' keyword
    .disable_symbol("+=");      // disable the '+=' operator

// The following all return parse errors.

engine.compile("let x = if true { 42 } else { 0 };")?;
//                      ^ 'if' is rejected as a reserved keyword

engine.compile("let x = 40 + 2; x += 1;")?;
//                                ^ '+=' is not recognized as an operator
//                         ^ other operators are not affected
```
