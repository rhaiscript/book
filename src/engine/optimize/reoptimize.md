Re-Optimize an AST
==================

{{#include ../../links.md}}

Sometimes it is more efficient to store one single, large script with delimited code blocks guarded by
constant variables.  This script is compiled once to an [`AST`].

Then, depending on the execution environment, constants are passed into the [`Engine`] and the [`AST`]
is _re_-optimized based on those constants via the `Engine::optimize_ast` method,
effectively pruning out unused code sections.

The final, optimized [`AST`] is then used for evaluations.

```rust , no_run
// Compile master script to AST
let master_ast = engine.compile(
"
    if SCENARIO == 1 {
        do_work();
    } else if SCENARIO == 2 {
        do_something();
    } else if SCENARIO == 3 {
        do_something_else();
    } else {
        do_nothing();
    }
")?;

// Create a new 'Scope' - put constants in it to aid optimization
let mut scope = Scope::new();
scope.push_constant("SCENARIO", 1_i64);

// Re-optimize the AST
let new_ast = engine.optimize_ast(&scope, master_ast.clone(), OptimizationLevel::Simple);

// 'new_ast' is essentially: 'do_work()'
```
