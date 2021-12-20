Variable Resolver
=================

{{#include ../links.md}}

By default, Rhai looks up access to [variables] from the enclosing block scope,
working its way outwards until it reaches the top (global) level, then it
searches the [`Scope`] that is passed into the `Engine::eval` call.

There is a built-in facility for advanced users to _hook_ into the [variable]
resolution service and to override its default behavior.

To do so, provide a closure to the [`Engine`] via the `Engine::on_var` method:

```rust no_run
let mut engine = Engine::new();

// Register a variable resolver.
engine.on_var(|name, index, context| {
    match name {
        "MYSTIC_NUMBER" => Ok(Some(42_i64.into())),
        // Override a variable - make it not found even if it exists!
        "DO_NOT_USE" => Err(EvalAltResult::ErrorVariableNotFound(name.to_string(), Position::NONE).into()),
        // Silently maps 'chameleon' into 'innocent'.
        "chameleon" => context.scope().get_value("innocent").map(Some).ok_or_else(|| 
            EvalAltResult::ErrorVariableNotFound(name.to_string(), Position::NONE).into()
        ),
        // Return Ok(None) to continue with the normal variable resolution process.
        _ => Ok(None)
    }
});
```


Returned Values are Constants
----------------------------

[Variable] values, if any returned, are treated as _constants_ by the script and cannot be assigned to.
This is to avoid needing a mutable reference to the underlying data provider which may not be possible to obtain.

In order to change these [variables], it is best to push them into a custom [`Scope`] instead of using
a variable resolver. Then these [variables] can be assigned to and their updated values read back after
the script is evaluated.


Benefits of Using a Variable Resolver
------------------------------------

1. Avoid having to maintain a custom [`Scope`] with all [variables] regardless of need (because a script may not use them all).

2. _Short-circuit_ [variable] access, essentially overriding standard behavior.

3. _Lazy-load_ [variables] when they are accessed, not up-front. This benefits when the number of [variables] is very large, when they are timing-dependent, or when they are expensive to load.

4. Rename system [variables] on a script-by-script basis without having to construct different [`Scope`]'s.


Function Signature
------------------

The function signature passed to `Engine::on_var` takes the following form:

> `Fn(name: &str, index: usize, context: &EvalContext)`  
> &nbsp;&nbsp;&nbsp;&nbsp;`-> Result<Option<Dynamic>, Box<EvalAltResult>> + 'static`

where:

| Parameter |      Type      | Description                                                                                                                                                                                                                                                                                                                                                                                          |
| --------- | :------------: | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `name`    |     `&str`     | [variable] name                                                                                                                                                                                                                                                                                                                                                                                      |
| `index`   |    `usize`     | an offset from the bottom of the current [`Scope`] that the [variable] is supposed to reside.<br/>Offsets start from 1, with 1 meaning the last [variable] in the current [`Scope`].  Essentially the correct [variable] is at position `scope.len() - index`.<br/>If `index` is zero, then there is no pre-calculated offset position and a search through the current [`Scope`] must be performed. |
| `context` | `&EvalContext` | the current _evaluation context_                                                                                                                                                                                                                                                                                                                                                                     |

and `EvalContext` is a type that encapsulates the current _evaluation context_ and exposes the following:

| Method              |               Return type               | Description                                                                                                                                     |
| ------------------- | :-------------------------------------: | ----------------------------------------------------------------------------------------------------------------------------------------------- |
| `scope()`           |                `&Scope`                 | reference to the current [`Scope`]                                                                                                              |
| `engine()`          |                `&Engine`                | reference to the current [`Engine`]                                                                                                             |
| `source()`          |             `Option<&str>`              | reference to the current source, if any                                                                                                         |
| `iter_imports()`    | `impl Iterator<Item = (&str, &Module)>` | iterator of the current stack of [modules] imported via `import` statements                                                                     |
| `imports()`         |               `&Imports`                | reference to the current stack of [modules] imported via `import` statements; requires the [`internals`] feature                                |
| `iter_namespaces()` |     `impl Iterator<Item = &Module>`     | iterator of the [namespaces][function namespaces] (as [modules]) containing all script-defined [functions]                                      |
| `namespaces()`      |              `&[&Module]`               | reference to the [namespaces][function namespaces] (as [modules]) containing all script-defined [functions]; requires the [`internals`] feature |
| `this_ptr()`        |           `Option<&Dynamic>`            | reference to the current bound [`this`] pointer, if any                                                                                         |
| `call_level()`      |                 `usize`                 | the current nesting level of function calls                                                                                                     |

### Return Value

The return value is `Result<Option<Dynamic>, Box<EvalAltResult>>` where:

| Value                     | Description                                                                                                                                                                                                              |
| ------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `Ok(None)`                | normal [variable] resolution process should continue, i.e. continue searching through the [`Scope`]                                                                                                                      |
| `Ok(Some(Dynamic))`       | value of the [variable], treated as a constant                                                                                                                                                                           |
| `Err(Box<EvalAltResult>)` | error that is reflected back to the [`Engine`].<br/>Normally this is `EvalAltResult::ErrorVariableNotFound(var_name, Position::NONE)` to indicate that the [variable] does not exist, but it can be any `EvalAltResult`. |
