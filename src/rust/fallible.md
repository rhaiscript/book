Register a Fallible Rust Function
================================

{{#include ../links.md}}

If a function is _fallible_ (i.e. it returns a `Result<_, _>`), it can be registered with via
`Engine::register_result_fn`.

The function must return `Result<T, Box<EvalAltResult>>` where `T` is any clonable type.

```rust no_run
use rhai::{Engine, EvalAltResult, Position};

// Function that may fail - the error type must be 'Box<EvalAltResult>'
fn safe_divide(x: i64, y: i64) -> Result<i64, Box<EvalAltResult>> {
    if y == 0 {
        // Return an error if y is zero
        Err("Division by zero!".into())         // shortcut to create Box<EvalAltResult::ErrorRuntime>
    } else {
        Ok(x / y)
    }
}

let mut engine = Engine::new();

// Fallible functions that return Result values must use register_result_fn()
engine.register_result_fn("divide", safe_divide);

if let Err(error) = engine.eval::<i64>("divide(40, 0)") {
    println!("Error: {:?}", *error);         // prints ErrorRuntime("Division by zero detected!", (1, 1)")
}
```

Create a `Box<EvalAltResult>`
----------------------------

`Box<EvalAltResult>` implements `From<&str>` and `From<String>` etc.
and the error text gets converted into `Box<EvalAltResult::ErrorRuntime>`.

The error values are `Box`-ed in order to reduce memory footprint of the error path,
which should be hit rarely.
