Compile a Script (to AST)
========================

{{#include ../links.md}}

To repeatedly evaluate a script, _compile_ it first with `Engine::compile` into an `AST`
(**A**bstract **S**yntax **T**ree) form.

`Engine::eval_ast_XXX` and `Engine::run_ast_XXX` evaluate a pre-compiled `AST`.

```rust,no_run
// Compile to an AST and store it for later evaluations
let ast = engine.compile("40 + 2")?;

for _ in 0..42 {
    let result: i64 = engine.eval_ast(&ast)?;

    println!("Answer #{}: {}", i, result);      // prints 42
}
```

~~~admonish tip.small "Tip: Compile script file"

Compiling script files is also supported via `Engine::compile_file`
(not available for [`no_std`] or [WASM] builds).

```rust,no_run
let ast = engine.compile_file("hello_world.rhai".into())?;
```
~~~

~~~admonish info.small "`AST` manipulation API"

Advanced users who may want to manipulate an `AST`, especially the functions contained within,
should see the section on [_Manage AST's_](ast.md) for more details.
~~~
