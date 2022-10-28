Register a Fallible Rust Function
=================================

{{#include ../links.md}}

~~~admonish tip.side.wide "Tip: Consider `Dynamic`"

A lot of times it is not necessary to register fallible functions.

Simply have the function returns [`Dynamic`].
Upon error, return [`()`] which is idiomatic in Rhai.

See [here](dynamic-return.md) for more details.
~~~

If a function is _fallible_ (i.e. it returns a `Result<_, _>`), it can also be registered with via
`Engine::register_fn`.

The function must return `Result<T, Box<EvalAltResult>>` where `T` is any clonable type.

In other words, the error type _must_ be `Box<EvalAltResult>`.  It is `Box`ed in order to reduce
the size of the `Result` type since the error path is rarely hit.

```rust
use rhai::{Engine, EvalAltResult};

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

engine.register_fn("divide", safe_divide);

if let Err(error) = engine.eval::<i64>("divide(40, 0)") {
    println!("Error: {error:?}");               // prints ErrorRuntime("Division by zero detected!", (1, 1)")
}
```


~~~admonish tip.small "Tip: Create a `Box<EvalAltResult>`"

`Box<EvalAltResult>` implements `From<&str>` and `From<String>` etc.
and the error text gets converted into `Box<EvalAltResult::ErrorRuntime>`.

The error values are `Box`-ed in order to reduce memory footprint of the error path,
which should be hit rarely.
~~~
