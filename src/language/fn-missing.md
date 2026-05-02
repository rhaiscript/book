Missing Function Handling
=========================

{{#include ../links.md}}

[`Engine::on_missing_function`]: https://docs.rs/rhai/{{version}}/rhai/struct.Engine.html#method.on_missing_function


~~~admonish warning.small "Requires `internals`"

This is an advanced feature that requires the [`internals`] feature to be enabled.
~~~

Normally, when a [function] or a [method] is called but not found by the [`Engine`],
the script terminates with an error.

It is possible to completely control this behavior via a special callback function
registered into an [`Engine`] via `on_missing_function`.

Using this callback, for instance, it is simple to instruct Rhai to return a default value, call a
fallback [function] on the fly, or even modify the error returned, when a non-existent [function] or
[method] is called.


Function Signature
------------------

The function signature passed to [`Engine::on_missing_function`] takes the following form.

> ```rust
> Fn(name: &str, args: &mut [&mut Dynamic], is_method_call: bool, context: EvalContext)  
>       -> Result<Option<Dynamic>, Box<EvalAltResult>>
> ```

where:

| Parameter        |         Type          | Description                                                                                                           |
| ---------------- | :-------------------: | --------------------------------------------------------------------------------------------------------------------- |
| `name`           |        `&str`         | name of the [function] called                                                                                         |
| `args`           | `&mut [&mut Dynamic]` | arguments passed to the [function]                                                                                    |
| `is_method_call` |        `bool`         | `true` if the [function] is called as a [method], which means that the first argument (the object) cannot be consumed |
| `context`        |    [`EvalContext`]    | the current _evaluation context_                                                                                      |

### Return value

Return `Some` as the value returned by the [function] call, or `None` to return the standard error.


Example
-------

```rust
engine.on_missing_function(|name, args, is_method_call, context| {
    if is_method_call && name == "greet" {
        Ok(Some(Dynamic::from("hello")))
    } else {
        Ok(None)    // fall through to standard error
    }
});
```
