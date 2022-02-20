Re-Optimize an AST
==================

{{#include ../../links.md}}

Sometimes it is more efficient to store one single, large script with delimited code blocks guarded by
constant variables.  This script is compiled once to an [`AST`].

Then, depending on the execution environment, constants are passed into the [`Engine`] and the
[`AST`] is _re_-optimized based on those constants via `Engine::optimize_ast`, effectively pruning
out unused code sections.

The final, optimized [`AST`] is then used for evaluations.

```rust,no_run
// Compile master script to AST
let master_ast = engine.compile(
"
    fn do_work() {
        // Constants in scope are also propagated into functions
        print(SCENARIO);
    }

    switch SCENARIO {
        1 => do_work(),
        2 => do_something(),
        3 => do_something_else(),
        _ => do_nothing()
    }
")?;

for n in 0..5_i64 {
    // Create a new scope - put constants in it to aid optimization
    let mut scope = Scope::new();
    scope.push_constant("SCENARIO", n);

    // Re-optimize the AST
    let new_ast = engine.optimize_ast(&scope, master_ast.clone(), OptimizationLevel::Simple);

    // Run it
    engine.run_ast(&new_ast)?;
}
```

```admonish note "Constants propagation"

Beware that [constants] inside the custom [`Scope`] will also be propagated to [functions] defined
within the script while normally such [functions] are _pure_ and cannot see [variables]/[constants]
within the global [`Scope`].
```
