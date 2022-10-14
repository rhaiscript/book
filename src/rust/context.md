`NativeCallContext`
===================

{{#include ../links.md}}

If the _first_ parameter of a function is of type `rhai::NativeCallContext`, then it is treated
specially by the [`Engine`].

`NativeCallContext` is a type that encapsulates the current _call context_ of a Rust function call
and exposes the following.

| Method                    |                      Return type                       | Description                                                                                                                                                                                                                                |
| ------------------------- | :----------------------------------------------------: | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `engine()`                |                 [`&Engine`][`Engine`]                  | the current [`Engine`], with all configurations and settings.<br/>This is sometimes useful for calling a script-defined function within the same evaluation context using [`Engine::call_fn`][`call_fn`], or calling a [function pointer]. |
| `fn_name()`               |                         `&str`                         | name of the function called (useful when the same Rust function is mapped to multiple Rhai-callable function names)                                                                                                                        |
| `source()`                |                     `Option<&str>`                     | reference to the current source, if any                                                                                                                                                                                                    |
| `position()`              |                       `Position`                       | position of the function call                                                                                                                                                                                                              |
| `call_level()`            |                        `usize`                         | the current nesting level of function calls                                                                                                                                                                                                |
| `tag()`                   |                [`&Dynamic`][`Dynamic`]                 | reference to the custom state that is persistent during the current run                                                                                                                                                                    |
| `iter_imports()`          | `impl Iterator<Item = (&str,`[`&Module`][`Module`]`)>` | iterator of the current stack of [modules] imported via `import` statements, in reverse order (i.e. later [modules] come first)                                                                                                            |
| `global_runtime_state()`  |     [`&GlobalRuntimeState`][`GlobalRuntimeState`]      | reference to the current [global runtime state][`GlobalRuntimeState`] (including the stack of [modules] imported via `import` statements); requires the [`internals`] feature                                                              |
| `iter_namespaces()`       |     `impl Iterator<Item =`[`&Module`][`Module`]`>`     | iterator of the [namespaces][function namespaces] (as [modules]) containing all script-defined [functions], in reverse order (i.e. later [modules] come first)                                                                             |
| `namespaces()`            |                [`&[&Module]`][`Module`]                | reference to the [namespaces][function namespaces] (as [modules]) containing all script-defined [functions]; requires the [`internals`] feature                                                                                            |
| `call_fn(...)`            |            `Result<T, Box<EvalAltResult>>`             | call a function with the supplied arguments, casting the result into the required type                                                                                                                                                     |
| `call_native_fn(...)`     |            `Result<T, Box<EvalAltResult>>`             | call a registered native Rust function with the supplied arguments, casting the result into the required type                                                                                                                              |
| `call_fn_raw(...)`        |      `Result<`[`Dynamic`]`, Box<EvalAltResult>>`       | call a function with the supplied arguments; this is an advanced method                                                                                                                                                                    |
| `call_native_fn_raw(...)` |      `Result<`[`Dynamic`]`, Box<EvalAltResult>>`       | call a registered native Rust function with the supplied arguments; this is an advanced method                                                                                                                                             |


~~~admonish example "Example &ndash; Implement Safety Checks"

The native call context is useful for protecting a function from malicious scripts.

```rust
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
~~~

~~~admonish example "Example &ndash; Call a Function Within a Function"

The _native call context_ can be used to call a [function] within the current evaluation
via `call_fn` (or more commonly `call_native_fn`).

```rust
use rhai::{Engine, NativeCallContext};

let mut engine = Engine::new();

// A function expecting a callback in form of a function pointer.
engine.register_fn("super_call", |context: NativeCallContext, value: i64| {
    // Call a function within the current evaluation!
    // 'call_native_fn' ensures that only registered native Rust functions
    // are called, so a scripted function named 'double' cannot hijack
    // the process.
    // To also include scripted functions, use 'call_fn' instead.
    context.call_native_fn::<i64>("double", (value,))
    //                                      ^^^^^^^^ arguments passed in tuple
});
```
~~~

~~~admonish example "Example &ndash; Implement a Callback"

The _native call context_ can be used to call a [function pointer] or [closure] that has been passed
as a parameter to the function (via `FnPtr::call_within_context`), thereby implementing a _callback_.

```rust
use rhai::{Dynamic, FnPtr, NativeCallContext, EvalAltResult};

pub fn greet(context: NativeCallContext, callback: FnPtr) -> Result<String, Box<EvalAltResult>>
{
    // Call the callback closure with the current evaluation context!
    let name = callback.call_within_context(&context, ())?;
    Ok(format!("hello, {}!", name))
}
```
~~~


Advanced Usage &ndash; Recreate `NativeCallContext`
---------------------------------------------------

The `NativeCallContext` type encapsulates the entire _context_ of a script up to the
particular point of the native Rust function call.

The fields of `NativeCallContext` can be individually stored (e.g. `&str` fields can be stored as
`String`'s) for later use, when a new `NativeCallContext` can be constructed based on these stored
fields.

A reconstructed `NativeCallContext` acts almost the same as the original instance, so it is possible
to suspend the evaluation of a script, and to continue at a later time with a new
`NativeCallContext`.

Doing so requires the [`internals`] feature to access internal API's.

### Step 1: Store all fields

```rust
// Store fields for later use
let fn_name = context.fn_name().to_string();
let source = context.source().map(|s| s.to_string());
let global = context.global_runtime_state().unwrap().clone();
let pos = context.position();
let call_level = context.call_level();

// Store the paths of the stack of call modules up to this point.
let modules_list: Vec<String> = context.iter_namespaces()
                                       .map(|m| m.id().unwrap_or(""))
                                       .filter(|id| !id.is_empty())
                                       .map(|id| id.to_string())
                                       .collect();

// ... store the fields somewhere ...
```

### Step 2: Recreate `NativeCallContext`

```rust
// ... do something else ...

let mut libraries = Vec::<Shared<Module>>::new();

for path in modules_list {
    // Recreate the stack of call modules by resolving each path with
    // the module resolver.
    let module = engine.module_resolver().resolve(engine, None, &path, pos)?;

    libraries.push(module);
}

let lib: Vec<&Module> = libraries.iter().map(|m| m.as_ref()).collect();

let new_context = NativeCallContext::new_with_all_fields(
                        &engine,
                        &fn_name,
                        src,
                        &global,
                        &lib,
                        pos,
                        call_level
                  );
```
