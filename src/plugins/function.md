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

```rust,no_run
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

Non-pure functions, when passed a [constant] value as the first `&mut` parameter, will raise an
`EvalAltResult::ErrorAssignmentToConstant` error.

```rust,no_run
use rhai::plugin::*;        // a "prelude" import for macros

// This method is pure, so 'len' can be used on a constant 'MyType'.
#[export_fn(pure)]
pub fn len(my_type: &mut MyType) -> i64 {
    my_type.len()
}

// This method is not pure, so 'clear' will raise an error
// when used on a constant 'MyType'.
#[export_fn]
pub fn clear(my_type: &mut MyType) {
    my_type.clear();
}
```


Fallible Functions
------------------

To register [fallible functions] (i.e. functions that may return errors), apply the
`#[export_fn(return_raw)]` attribute on plugin functions that return `Result<Dynamic, Box<EvalAltResult>>`.

A syntax error is generated if the function with `#[export_fn(return_raw)]` does not
have the appropriate return type.

```rust,no_run
use rhai::plugin::*;        // a "prelude" import for macros

#[export_fn(return_raw)]
pub fn double_and_divide(x: i64, y: i64) -> Result<Dynamic, Box<EvalAltResult>> {
    if y == 0 {
        Err("Division by zero!".into())
    } else {
        let result = (x * 2) / y;
        Ok(result.into())
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

If the _first_ parameter of a function is of type `rhai::NativeCallContext`, then it is treated
specially by the plugins system.

`NativeCallContext` is a type that encapsulates the current _native call context_ and exposes the following:

| Field               |                  Type                   | Description                                                                                                                                                                                                                                |
| ------------------- | :-------------------------------------: | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `engine()`          |                `&Engine`                | the current [`Engine`], with all configurations and settings.<br/>This is sometimes useful for calling a script-defined function within the same evaluation context using [`Engine::call_fn`][`call_fn`], or calling a [function pointer]. |
| `fn_name()`         |                 `&str`                  | name of the function called (useful when the same Rust function is mapped to multiple Rhai-callable function names)                                                                                                                        |
| `source()`          |             `Option<&str>`              | reference to the current source, if any                                                                                                                                                                                                    |
| `iter_imports()`    | `impl Iterator<Item = (&str, &Module)>` | iterator of the current stack of [modules] imported via `import` statements                                                                                                                                                                |
| `imports()`         |               `&Imports`                | reference to the current stack of [modules] imported via `import` statements; requires the [`internals`] feature                                                                                                                           |
| `iter_namespaces()` |     `impl Iterator<Item = &Module>`     | iterator of the namespaces (as [modules]) containing all script-defined functions                                                                                                                                                          |
| `namespaces()`      |              `&[&Module]`               | reference to the namespaces (as [modules]) containing all script-defined functions; requires the [`internals`] feature                                                                                                                     |

This first parameter, if exists, will be stripped before all other processing.  It is _virtual_.
Most importantly, it does _not_ count as a parameter to the function and there is no need to provide
this argument when calling the function in Rhai.

The native call context can be used to call a [function pointer] or [closure] that has been passed
as a parameter to the function, thereby implementing a _callback_:

```rust,no_run
use rhai::{Dynamic, FnPtr, NativeCallContext, EvalAltResult};
use rhai::plugin::*;        // a "prelude" import for macros

#[export_fn(return_raw)]
pub fn greet(context: NativeCallContext, callback: FnPtr)
                            -> Result<Dynamic, Box<EvalAltResult>>
{
    // Call the callback closure with the current context
    // to obtain the name to greet!
    let name = callback.call_dynamic(context, None, [])?;
    Ok(format!("hello, {}!", name).into())
}
```

The native call context is also useful in another scenario: protecting a function from malicious scripts.

```rust,no_run
use rhai::{Dynamic, Array, NativeCallContext, EvalAltResult, Position};
use rhai::plugin::*;        // a "prelude" import for macros

// This function builds an array of arbitrary size, but is protected
// against attacks by first checking with the allowed limit set
// into the 'Engine'.
#[export_fn(return_raw)]
pub fn grow(context: NativeCallContext, size: i64)
                            -> Result<Dynamic, Box<EvalAltResult>>
{
    // Make sure the function does not generate a
    // data structure larger than the allowed limit
    // for the Engine!
    if size as usize > context.engine().max_array_size()
    {
        return EvalAltResult::ErrorDataTooLarge(
            "Size to grow".to_string(),
            context.engine().max_array_size(),
            size as usize,
            Position::NONE,
        ).into();
    }

    let array = Array::new();

    for x in 0..size {
        array.push(x.into());
    }

    OK(array.into())
}
```
