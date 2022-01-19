Your First Script in Rhai
=========================

{{#include ../links.md}}


Run a Script
------------

To get going with Rhai is as simple as creating an instance of the scripting engine `rhai::Engine`
via `Engine::new`, then calling `Engine::run`.

```rust,no_run
use rhai::{Engine, EvalAltResult};

pub fn main() -> Result<(), Box<EvalAltResult>>
//                          ^^^^^^^^^^^^^^^^^^
//                          Rhai API error type
{
    // Create an 'Engine'
    let engine = Engine::new();

    // Your first Rhai Script
    let script = "print(40 + 2);";

    // Run the script - prints "42"
    engine.run(script)?;

    // Done!
    Ok(())
}
```


Get a Return Value
------------------

To return a value from the script, use `Engine::eval` instead.

```rust,no_run
use rhai::{Engine, EvalAltResult};

pub fn main() -> Result<(), Box<EvalAltResult>>
{
    let engine = Engine::new();

    let result = engine.eval::<i64>("40 + 2")?;
    //                      ^^^^^^^ required: cast the result to a type

    println!("Answer: {}", result);             // prints 42

    Ok(())
}
```


Use Script Files
----------------

Or evaluate a script file directly with `Engine::run_file` or `Engine::eval_file`
(not available under [`no_std`] or in [WASM] builds).

```rust,no_run
let result = engine.eval_file::<i64>("hello_world.rhai".into())?;
//                                   ^^^^^^^^^^^^^^^^^^^^^^^^^
//                                   a 'PathBuf' is needed

// Running a script file also works in a similar manner
engine.run_file("hello_world.rhai".into())?;
```

Rhai script files are customarily named with the extension `.rhai`.


Specify the Return Type
-----------------------

The type parameter for `Engine::eval_XXX` methods is used to specify the type of the return value,
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


Unix Shebangs in Script Files
----------------------------

On Unix-like systems, the _shebang_ (`#!`) is used at the very beginning of a script file to mark a
script with an interpreter (for Rhai this would be [`rhai-run`]({{rootUrl}}/start/bin.md)).

If a script file starts with `#!`, the entire first line is skipped by `Engine::compile_file` and
`Engine::eval_file`. Because of this, Rhai scripts with shebangs at the beginning need no special processing.

This behavior is also present for non-Unix (e.g. Windows) environments.

```js
#!/home/to/me/bin/rhai-run

// This is a Rhai script

let answer = 42;
print(`The answer is: ${answer}`);
```
