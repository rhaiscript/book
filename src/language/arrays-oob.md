Out-of-Bounds Index for Arrays
==============================

{{#include ../links.md}}

[`Engine::on_invalid_array_index`]: https://docs.rs/rhai/{{version}}/rhai/struct.Engine.html#method.on_invalid_array_index
[`Target`]: https://docs.rs/rhai/latest/rhai/enum.Target.html


~~~admonish warning.small "Requires `internals`"

This is an advanced feature that requires the [`internals`] feature to be enabled.
~~~

Normally, when an index is out-of-bounds for an [array], an error is raised.

It is possible to completely control this behavior via a special callback function
registered into an [`Engine`] via `on_invalid_array_index`.

Using this callback, for instance, it is simple to instruct Rhai to extend the [array] to
accommodate this new element, or to return a default value instead of raising an error.


Function Signature
------------------

The function signature passed to [`Engine::on_invalid_array_index`] takes the following form.

> ```rust
> Fn(array: &mut Array, index: i64, context: EvalContext) -> Result<Target, Box<EvalAltResult>>
> ```

where:

| Parameter |         Type          | Description                      |
| --------- | :-------------------: | -------------------------------- |
| `array`   | [`&mut Array`][array] | the [array] being accessed       |
| `index`   |         `i64`         | index value                      |
| `context` |    [`EvalContext`]    | the current _evaluation context_ |

### Return value

The return value is `Result<Target, Box<EvalAltResult>>`.

[`Target`] is an advanced type, available only under the [`internals`] feature, that represents a
_reference_ to a [`Dynamic`] value.

It can be used to point to a particular value within the [array] or a new temporary value.


Example
-------

```rust
engine.on_invalid_array_index(|arr, index, _| {
    match index {
        -100 => {
            // The array can be modified in place
            arr.push((42_i64).into());

            // Return a mutable reference to an element
            let value_ref = arr.last_mut().unwrap();
            Ok(value_ref.into())
        }
        100 => {
            // Return a temporary value (not a reference)
            let value = Dynamic::from(100_i64);
            Ok(value.into())
        }
        // Return the standard out-of-bounds error
        _ => Err(EvalAltResult::ErrorArrayBounds(
                arr.len(), index, Position::NONE
             ).into()),
    }
});
```
