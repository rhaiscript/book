`EvalContext`
=============

{{#include ../links.md}}

```admonish info.side.wide "Usage"

Many functions in advanced API's contain an `EvalContext` parameter in order to allow
the current evaluation state to be accessed and/or modified.
```

`EvalContext` is a type that encapsulates the current _evaluation context_ and exposes the following methods.

| Method                       |                      Return type                       | Description                                                                                                                                                             |
| ---------------------------- | :----------------------------------------------------: | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `scope()`                    |                  [`&Scope`][`Scope`]                   | reference to the current [`Scope`]                                                                                                                                      |
| `scope_mut()`                |              [`&mut &mut Scope`][`Scope`]              | mutable reference to the current [`Scope`]; [variables] can be added to/removed from it                                                                                 |
| `engine()`                   |                 [`&Engine`][`Engine`]                  | reference to the current [`Engine`]                                                                                                                                     |
| `source()`                   |                     `Option<&str>`                     | reference to the current source, if any                                                                                                                                 |
| `tag()`                      |                [`&Dynamic`][`Dynamic`]                 | reference to the custom state that is persistent during the current run                                                                                                 |
| `tag_mut()`                  |              [`&mut Dynamic`][`Dynamic`]               | mutable reference to the custom state that is persistent during the current run                                                                                         |
| `iter_imports()`             | `impl Iterator<Item = (&str,`[`&Module`][`Module`]`)>` | iterator of the current stack of [modules] imported via [`import`] statements, in reverse order (i.e. later [modules] come first); not available under [`no_module`]    |
| `global_runtime_state()`     |     [`&GlobalRuntimeState`][`GlobalRuntimeState`]      | reference to the current [global runtime state][`GlobalRuntimeState`] (including the stack of [modules] imported via [`import`] statements)                             |
| `global_runtime_state_mut()` | [`&mut &mut GlobalRuntimeState`][`GlobalRuntimeState`] | mutable reference to the current [global runtime state][`GlobalRuntimeState`]; use this to access the [`debugger`][debugger] field in order to set/clear [break-points] |
| `iter_namespaces()`          |     `impl Iterator<Item =`[`&Module`][`Module`]`>`     | iterator of the [namespaces][function namespaces] (as [modules]) containing all script-defined [functions], in reverse order (i.e. later [modules] come first)          |
| `namespaces()`               |                [`&[&Module]`][`Module`]                | reference to the [namespaces][function namespaces] (as [modules]) containing all script-defined [functions]                                                             |
| `this_ptr()`                 |            [`Option<&Dynamic>`][`Dynamic`]             | reference to the current bound `this` pointer, if any                                                                                                                   |
| `call_level()`               |                        `usize`                         | the current nesting level of [function] calls                                                                                                                           |
