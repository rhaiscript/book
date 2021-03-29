Hello World in Rhai
===================

{{#include ../links.md}}

To get going with Rhai is as simple as creating an instance of the scripting engine `rhai::Engine` via
`Engine::new`, then calling the `eval` method:

```rust,no_run
use rhai::{Engine, EvalAltResult};

fn main() -> Result<(), Box<EvalAltResult>>
{
    let engine = Engine::new();

    let result = engine.eval::<i64>("40 + 2")?;
    //                      ^^^^^^^ cast the result to an 'i64', this is required

    println!("Answer: {}", result);             // prints 42

    Ok(())
}
```

Evaluate a script file directly:

```rust,no_run
// 'eval_file' takes a 'PathBuf'
let result = engine.eval_file::<i64>("hello_world.rhai".into())?;
```


Error Type
----------

`rhai::EvalAltResult` is the standard Rhai error type, which is a Rust `enum` containing all errors encountered
during the parsing or evaluation process.


Return Type
-----------

The type parameter for `Engine::eval` is used to specify the type of the return value,
which _must_ match the actual type or an error is returned. Rhai is very strict here.

There are two ways to specify the return type &ndash; _turbofish_ notation, or type inference.

Use [`Dynamic`] for uncertain return types.

```rust,no_run
let result = engine.eval::<i64>("40 + 2")?;     // return type is i64, specified using 'turbofish' notation

let result: i64 = engine.eval("40 + 2")?;       // return type is inferred to be i64

result.is::<i64>() == true;

let result: Dynamic = engine.eval("boo()")?;    // use 'Dynamic' if you're not sure what type it'll be!

let result = engine.eval::<String>("40 + 2")?;  // returns an error because the actual return type is i64, not String
```


Unix Shebangs
-------------

On Unix-like systems, the _shebang_ (`#!`) is used at the very beginning of a script file to mark a
script with an interpreter (for Rhai this would be [`rhai-run`]({{rootUrl}}/start/bin.md)).

If a script file starts with `#!`, the entire first line is skipped by `Engine::compile_file` and
`Engine::eval_file`. Because of this, Rhai scripts with shebangs at the beginning need no special processing.

```rust,no_run
#!/home/to/me/bin/rhai-run

// This is a Rhai script

print("The answer is: " + 42);
```
