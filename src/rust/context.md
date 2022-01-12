`NativeCallContext`
===================

{{#include ../links.md}}


If the _first_ parameter of a function is of type `rhai::NativeCallContext`, then it is treated
specially by the [`Engine`].

`NativeCallContext` is a type that encapsulates the current _call context_ of a Rust function call
and exposes the following.

| Method                   |               Return type               | Description                                                                                                                                                                                                                                |
| ------------------------ | :-------------------------------------: | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `engine()`               |                `&Engine`                | the current [`Engine`], with all configurations and settings.<br/>This is sometimes useful for calling a script-defined function within the same evaluation context using [`Engine::call_fn`][`call_fn`], or calling a [function pointer]. |
| `fn_name()`              |                 `&str`                  | name of the function called (useful when the same Rust function is mapped to multiple Rhai-callable function names)                                                                                                                        |
| `source()`               |             `Option<&str>`              | reference to the current source, if any                                                                                                                                                                                                    |
| `position()`             |               `Position`                | position of the function call                                                                                                                                                                                                              |
| `iter_imports()`         | `impl Iterator<Item = (&str, &Module)>` | iterator of the current stack of [modules] imported via `import` statements, in reverse order (i.e. later [modules] come first)                                                                                                            |
| `global_runtime_state()` |          `&GlobalRuntimeState`          | reference to the current global runtime state (including the stack of [modules] imported via `import` statements); requires the [`internals`] feature                                                                                      |
| `iter_namespaces()`      |     `impl Iterator<Item = &Module>`     | iterator of the [namespaces][function namespaces] (as [modules]) containing all script-defined [functions], in reverse order (i.e. later [modules] come first)                                                                             |
| `namespaces()`           |              `&[&Module]`               | reference to the [namespaces][function namespaces] (as [modules]) containing all script-defined [functions]; requires the [`internals`] feature                                                                                            |
| `call_fn()`              |     `Result<T, Box<EvalAltResult>>`     | call a function with the supplied arguments, casting the result into the required type                                                                                                                                                     |
| `call_fn_raw()`          |  `Result<Dynamic, Box<EvalAltResult>>`  | call a function with the supplied arguments; this is an advanced method                                                                                                                                                                    |


Implement Safety Checks
-----------------------

The native call context is useful for protecting a function from malicious scripts.

```rust,no_run
use rhai::{Array, NativeCallContext, EvalAltResult, Position};

// This function builds an array of arbitrary size, but is protected
// against attacks by first checking with the allowed limit set
// into the 'Engine'.
pub fn grow(context: NativeCallContext, size: i64) -> Result<Array, Box<EvalAltResult>>
{
    // Make sure the function does not generate a
    // data structure larger than the allowed limit
    // for the Engine!
    if size as usize > context.engine().max_array_size() {
        return Err(EvalAltResult::ErrorDataTooLarge(
            "Size to grow".to_string(),
            context.engine().max_array_size(),
            size as usize,
            context.position(),
        ).into());
    }

    let array = Array::new();

    for x in 0..size {
        array.push(x.into());
    }

    OK(array)
}
```


Call a Function within a Function
---------------------------------

The _native call context_ can be used to call a [function] within the current evaluation
via `call_fn`.

```rust,no_run
use rhai::{Engine, NativeCallContext};

let mut engine = Engine::new();

// A function expecting a callback in form of a function pointer.
fn super_call(context: NativeCallContext, value: i64) -> Result<i64, Box<EvalAltResult>>
{
    // Use 'call_fn' to call a function within the current evaluation!
    context.call_fn("double", (value,))
    //                        ^^^^^^^^ arguments passed in tuple
}

engine.register_result_fn("super_call", super_call);
```


Implement a Callback
--------------------

The _native call context_ can be used to call a [function pointer] or [closure] that has been passed
as a parameter to the function (via `FnPtr::call_with_context`), thereby implementing a _callback_.

```rust,no_run
use rhai::{Dynamic, FnPtr, NativeCallContext, EvalAltResult};

pub fn greet(context: NativeCallContext, callback: FnPtr) -> Result<String, Box<EvalAltResult>>
{
    // Call the callback closure with the current evaluation context!
    let name = callback.call_within_context(&context, ())?;
    Ok(format!("hello, {}!", name))
}
```
