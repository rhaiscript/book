Throw Exception on Error
=======================

{{#include ../links.md}}

All of [`Engine`]'s evaluation/consuming methods return `Result<T, Box<rhai::EvalAltResult>>`
with `EvalAltResult` holding error information.

To deliberately return an error during an evaluation, use the `throw` keyword.

```rust,no_run
if some_bad_condition_has_happened {
    throw error;    // 'throw' any value as the exception
}

throw;              // defaults to '()'
```

Exceptions thrown via `throw` in the script can be captured in Rust by matching
`Err(Box<EvalAltResult::ErrorRuntime(value, position)>)` with the exception value
captured by `value`.

```rust,no_run
let result = engine.eval::<i64>(r#"
    let x = 42;

    if x > 0 {
        throw x;
    }
"#);

println!("{}", result);     // prints "Runtime error: 42 (line 5, position 15)"
```


Catch a Thrown Exception
------------------------

It is possible to _catch_ an exception instead of having it abort the evaluation
of the entire script via the [`try` ... `catch`]({{rootUrl}}/language/try-catch.md)
statement common to many C-like languages.

```rust,no_run
try
{
    throw 42;
}
catch (err)         // 'err' captures the thrown exception value
{
    print(err);     // prints 42
}
```
