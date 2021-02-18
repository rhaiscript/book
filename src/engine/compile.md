Compile a Script (to AST)
========================

{{#include ../links.md}}

To repeatedly evaluate a script, _compile_ it first with `Engine::compile` into an `AST`
(abstract syntax tree) form.

`Engine::eval_ast` evaluates a pre-compiled `AST`.

```rust,no_run
// Compile to an AST and store it for later evaluations
let ast = engine.compile("40 + 2")?;

for _ in 0..42 {
    let result: i64 = engine.eval_ast(&ast)?;

    println!("Answer #{}: {}", i, result);      // prints 42
}
```

Compiling a script file is also supported with `Engine::compile_file`
(not available under [`no_std`] or in [WASM] builds):

```rust,no_run
let ast = engine.compile_file("hello_world.rhai".into())?;
```


`AST` Manipulation API
----------------------

Advanced users may want to manipulate an `AST`, especially the functions contained within.

See the section on [_Manage AST's_](ast.md) for more details.
