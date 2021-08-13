Export a Rust Function to Rhai
=============================

{{#include ../links.md}}


Sometimes only a few ad hoc functions are required and it is simpler to register
individual functions instead of a full-blown [plugin module].


Macros
------

| Macro                     | Signature                                                            | Description                                                                                         |
| ------------------------- | -------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------- |
| `#[export_fn]`            | apply to rust function defined in a Rust module                      | exports the function                                                                                |
| `register_exported_fn!`   | `register_exported_fn!(&mut `_engine_`, "`_name_`", `_function_`)`   | registers the function into an [`Engine`] under a specific name                                     |
| `set_exported_fn!`        | `set_exported_fn!(&mut `_module_`, "`_name_`", `_function_`)`        | registers the function into a [`Module`] under a specific name                                      |
| `set_exported_global_fn!` | `set_exported_global_fn!(&mut `_module_`, "`_name_`", `_function_`)` | registers the function into a [`Module`] under a specific name, exposing it to the global namespace |


`#[export_fn]` and `register_exported_fn!`
-----------------------------------------

Apply `#[export_fn]` onto a function defined at _module level_ to convert it into a Rhai plugin function.

The function cannot be nested inside another function &ndash; it can only be defined directly under a module.

To register the plugin function, simply call `register_exported_fn!`.  The name of the function can be
any text string, so it is possible to register _overloaded_ functions as well as operators.

```rust no_run
use rhai::plugin::*;        // import macros

#[export_fn]
fn increment(num: &mut i64) {
    *num += 1;
}

fn main() {
    let mut engine = Engine::new();

    // 'register_exported_fn!' registers the function as 'inc' with the Engine.
    register_exported_fn!(engine, "inc", increment);
}
```


Pure Functions
--------------

Some functions are _pure_ &ndash; i.e. they do not mutate any parameter, even though the first
parameter may be passed in as `&mut` (e.g. for a method function).

This is most commonly done to avoid expensive cloning for methods or [property
getters][getters/setters] that return information about a [custom type] and does not modify it.

Apply the `#[export_fn(pure)]` attribute on a plugin function to mark it as  _pure_.

Pure functions can be passed a [constant] value as the first `&mut` parameter.
The condition is that they _MUST NOT_ modify that value.

Non-pure functions, when passed a [constant] value as the first `&mut` parameter, will raise an
`EvalAltResult::ErrorAssignmentToConstant` error.

```rust no_run
use rhai::plugin::*;        // a "prelude" import for macros

// This method is pure, so 'len' can be used on a constant 'TestStruct'.
#[export_fn(pure)]
pub fn len(my_type: &mut TestStruct) -> i64 {
    my_type.len()
}

// This method is not pure, so 'clear' will raise an error
// when used on a constant 'TestStruct'.
#[export_fn]
pub fn clear(my_type: &mut TestStruct) {
    my_type.clear();
}
```


Fallible Functions
------------------

To register [fallible functions] (i.e. functions that may return errors), apply the
`#[export_fn(return_raw)]` attribute on plugin functions that return `Result<T, Box<EvalAltResult>>`
where `T` is any clonable type.

A syntax error is generated if the function with `#[export_fn(return_raw)]` does not
have the appropriate return type.

```rust no_run
use rhai::plugin::*;        // a "prelude" import for macros

#[export_fn(return_raw)]
pub fn double_and_divide(x: i64, y: i64) -> Result<i64, Box<EvalAltResult>> {
    if y == 0 {
        Err("Division by zero!".into())
    } else {
        Ok((x * 2) / y)
    }
}

fn main() {
    let mut engine = Engine::new();

    // Overloads the operator '+' with the Engine.
    register_exported_fn!(engine, "+", double_and_divide);
}
```


`NativeCallContext` Parameter
----------------------------

The _first_ parameter of a function can also be [`NativeCallContext`], which is treated
specially by the plugins system.
