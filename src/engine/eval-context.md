`EvalContext`
=============

{{#include ../links.md}}

Many functions in advanced APIs contain a parameter of type `EvalContext` in order to allow the
current evaluation state to be accessed and/or modified.

`EvalContext` encapsulates the current _evaluation context_ and exposes the following methods.

| Method                       |            Requires feature            |                      Return type                       | Description                                                                                                                                |
| ---------------------------- | :------------------------------------: | :----------------------------------------------------: | ------------------------------------------------------------------------------------------------------------------------------------------ |
| `engine()`                   |                                        |                 [`&Engine`][`Engine`]                  | reference to the current [`Engine`]                                                                                                        |
| `scope()`                    |                                        |                  [`&Scope`][`Scope`]                   | reference to the current [`Scope`]                                                                                                         |
| `scope_mut()`                |                                        |                [`&mut Scope`][`Scope`]                 | mutable reference to the current [`Scope`]; [variables] can be added to/removed from it                                                    |
| `source()`                   |                                        |                     `Option<&str>`                     | reference to the current source, if any                                                                                                    |
| `source_mut()`               |                                        |  [`&mut Option<ImmutableString>`][`ImmutableString`]   | mutable reference to the current source, if any                                                                                            |
| `tag()`                      |                                        |                [`&Dynamic`][`Dynamic`]                 | reference to the _custom state_ that is persistent during the current run                                                                  |
| `tag_mut()`                  |                                        |              [`&mut Dynamic`][`Dynamic`]               | mutable reference to the _custom state_ that is persistent during the current run                                                          |
| `iter_imports()`             |           not [`no_module`]            | `impl Iterator<Item = (&str,`[`&Module`][`Module`]`)>` | iterator of the current stack of [modules] imported via [`import`] statements, in reverse order (i.e. later imported [modules] come first) |
| `iter_namespaces()`          |          not [`no_function`]           |     `impl Iterator<Item =`[`&Module`][`Module`]`>`     | iterator of the [namespaces][function namespaces] (as [modules]) containing all script-defined [functions]                                 |
| `namespaces()`               | [`internals`],<br/>not [`no_function`] |            [`&[Shared<Module>]`][`Module`]             | reference to the [namespaces][function namespaces] (as shared [modules]) containing all script-defined [functions]                         |
| `namespaces_mut()`           | [`internals`],<br/>not [`no_function`] |              [`&mut [Module]`][`Module`]               | mutable reference to the [namespaces][function namespaces] (as shared [modules]) containing all script-defined [functions]                 |
| `this_ptr()`                 |                                        |            [`Option<&Dynamic>`][`Dynamic`]             | reference to the current bound `this` pointer, if any                                                                                      |
| `this_ptr_mut()`             |                                        |        [`&mut Option<&mut Dynamic>`][`Dynamic`]        | mutable reference to the current bound `this` pointer, if any                                                                              |
| `call_level()`               |                                        |                        `usize`                         | the current nesting level of [function] calls                                                                                              |
| `global_runtime_state()`     |             [`internals`]              |     [`&GlobalRuntimeState`][`GlobalRuntimeState`]      | reference to the current [global runtime state][`GlobalRuntimeState`]                                                                      |
| `global_runtime_state_mut()` |             [`internals`]              | [`&mut &mut GlobalRuntimeState`][`GlobalRuntimeState`] | mutable reference to the current [global runtime state][`GlobalRuntimeState`]                                                              |
| `debugger()`                 |      [`internals`], [`debugging`]      |      [`Option<&debugger::Debugger>`][`Debugger`]       | reference to the [debugging interface][debugger], if any                                                                                   |
| `debugger_mut()`             |      [`internals`], [`debugging`]      |    [`Option<&mut debugger::Debugger>`][`Debugger`]     | mutable reference to the [debugging interface][debugger], if any                                                                           |
| `new_frame()`                |                                        |                `EvalContextFrameGuard`                 | create a new frame that automatically restores field values upon `Drop`                                                                    |


New Frame
---------

The `EvalContext::new_frame()` method creates a new frame (via a _frame guard_) that automatically
restores field values when it is dropped. This is useful for temporarily modifying the `EvalContext`
and ensuring that it is properly restored afterwards, even if it exits due to an error.

The frame guard type returned by `EvalContext::new_frame()` is `EvalContextFrameGuard`,
supporting the following API:

| Method                     |            Requires feature            | Description                                                          |
| -------------------------- | :------------------------------------: | -------------------------------------------------------------------- |
| `with_new_caching_layer()` |                                        | push a new caching layer so that function resolutions do not persist |
| `rewind_scope(true/false)` |                                        | rewind (or not) the [scope][`Scope`] at the end of the frame         |
| `with_source(new_source)`  |                                        | modify the current source                                            |
| `clear_source()`           |                                        | clear the current source                                             |
| `with_tag(new_state)`      |                                        | modify the _custom state_                                            |
| `clear_tag()`              |                                        | clear the _custom state_ to [`()`]                                   |
| `with_import(module)`      |  [`internals`],<br/>not [`no_module`]  | push a new imported [module]                                         |
| `with_namespace(module)`   | [`internals`],<br/>not [`no_function`] | push a new [module] containing script-defined [functions]            |
| `up_call_level()`          |             [`internals`]              | increment the current nesting level of [function] calls              |


### Examples

The following pushes a new, empty, caching layer to make sure that new function resolutions
do not persist. This is useful if the resolution will be volatile.

```rust
// The resolution to function 'foo' will not be cached outside the frame.
// The next call to 'foo' will be resolved again instead of using the cached resolution,
// even if 'foo' is redefined.
let result: i64 = context.new_frame()
                         .with_new_caching_layer()
                         .call_fn("foo", (0_i64, true))?;
```

The following modifies the `EvalContext` before using, restoring its state afterwards.

```rust
{
     // Modify the context before using...
     let context = context.new_frame()
                          .rewind_scope(true)           // rewinds the scope...
                          .with_source("new source")    // new source...
                          .with_module(new_module)      // add new module...
                          .up_call_level();             // increment call level...
    //
    // Call a function with the modified context...
    let result: i64 = context.call_fn("foo", (0_i64, true))?;
    //
    // ... at end of block, context automatically restored to previous state.
}
```
