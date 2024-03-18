Non-Existent Property Handling for Object Maps
==============================================

{{#include ../links.md}}

[`Engine::on_map_missing_property`]: https://docs.rs/rhai/{{version}}/rhai/struct.Engine.html#method.on_map_missing_property
[`Target`]: https://docs.rs/rhai/latest/rhai/enum.Target.html


~~~admonish warning.small "Requires `internals`"

This is an advanced feature that requires the [`internals`] feature to be enabled.
~~~

Normally, when a property is accessed from an [object map] that does not exist, [`()`] is returned.
Via [`Engine:: set_fail_on_invalid_map_property`][options], it is possible to make this an error
instead.

Other than that, it is possible to completely control this behavior via a special callback function
registered into an [`Engine`] via `on_map_missing_property`.

Using this callback, for instance, it is simple to instruct Rhai to create a new property in the
[object map] on the fly, possibly with a default value, when a non-existent property is accessed.


Function Signature
------------------

The function signature passed to [`Engine::on_map_missing_property`] takes the following form.

> ```rust
> Fn(map: &mut Map, prop: &str, context: EvalContext) -> Result<Target, Box<EvalAltResult>>
> ```

where:

| Parameter |           Type           | Description                         |
| --------- | :----------------------: | ----------------------------------- |
| `map`     | [`&mut Map`][object map] | the [object map] being accessed     |
| `prop`    |          `&str`          | name of the property being accessed |
| `context` |     [`EvalContext`]      | the current _evaluation context_    |

### Return value

The return value is `Result<Target, Box<EvalAltResult>>`.

[`Target`] is an advanced type, available only under the [`internals`] feature, that represents a
_reference_ to a [`Dynamic`] value.

It can be used to point to a particular value within the [object map].


Example
-------

```rust
engine.on_map_missing_property(|map, prop, context| {
    match prop {
        "x" => {
            // The object-map can be modified in place
            map.insert("y".into(), (42_i64).into());

            // Return a mutable reference to an element
            let value_ref = map.get_mut("y").unwrap();
            Ok(value_ref.into())
        }
        "z" => {
            // Return a temporary value (not a reference)
            let value = Dynamic::from(100_i64);
            Ok(value.into())
        }
        // Return the standard property-not-found error
        _ => Err(EvalAltResult::ErrorPropertyNotFound(
                prop.to_string(), Position::NONE
             ).into()),
    }
});
```
