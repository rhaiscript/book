Variable Resolver
=================

{{#include ../links.md}}

By default, Rhai looks up access to [variables] from the enclosing block scope, working its way
outwards until it reaches the top (global) level, then it searches the [`Scope`] that is passed into
the `Engine::eval` call.

There is a built-in facility for advanced users to _hook_ into the [variable] resolution service and
to override its default behavior.

To do so, provide a closure to the [`Engine`] via `Engine::on_var`.

```rust
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

```admonish info.small "Benefits of using a variable resolver"

1. Avoid having to maintain a custom [`Scope`] with all [variables] regardless of need
   (because a script may not use them all).

2. _Short-circuit_ [variable] access, essentially overriding standard behavior.

3. _Lazy-load_ [variables] when they are accessed, not up-front.
   This benefits when the number of [variables] is very large, when they are timing-dependent,
   or when they are expensive to load.

4. Rename system [variables] on a script-by-script basis without having to construct different [`Scope`]'s.
```

```admonish warning.small "Returned values are constants"

[Variable] values returned by a variable resolver are treated as _[constants]_.

This is to avoid needing a mutable reference to the underlying data provider which may not be possible to obtain.

To change these [variables], better push them into a custom [`Scope`] instead of
using a variable resolver.
```

```admonish tip.small "Tip: Returning shared values"

It is possible to return a _shared_ value from a variable resolver.

This is one way to implement [Mutable Global State]({{rootUrl}}/patterns/global-mutable-state.md).
```


Function Signature
------------------

The function signature passed to `Engine::on_var` takes the following form.

> `Fn(name: &str, index: usize, context: EvalContext) -> Result<Option<Dynamic>, Box<EvalAltResult>>`

where:

| Parameter |      Type       | Description                                                                                                                                                                                                                                                                                                                                                                                          |
| --------- | :-------------: | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `name`    |     `&str`      | [variable] name                                                                                                                                                                                                                                                                                                                                                                                      |
| `index`   |     `usize`     | an offset from the bottom of the current [`Scope`] that the [variable] is supposed to reside.<br/>Offsets start from 1, with 1 meaning the last [variable] in the current [`Scope`].  Essentially the correct [variable] is at position `scope.len() - index`.<br/>If `index` is zero, then there is no pre-calculated offset position and a search through the current [`Scope`] must be performed. |
| `context` | [`EvalContext`] | mutable reference to the current _evaluation context_                                                                                                                                                                                                                                                                                                                                                |

and [`EvalContext`] is a type that encapsulates the current _evaluation context_.

### Return value

The return value is `Result<Option<Dynamic>, Box<EvalAltResult>>` where:

| Value                     | Description                                                                                                                                                                        |
| ------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Ok(None)`                | normal [variable] resolution process should continue, i.e. continue searching through the [`Scope`]                                                                                |
| `Ok(Some(value))`         | value (a [`Dynamic`]) of the [variable], treated as a [constant]                                                                                                                   |
| `Err(Box<EvalAltResult>)` | error that is reflected back to the [`Engine`], normally `EvalAltResult::ErrorVariableNotFound` to indicate that the [variable] does not exist, but it can be any `EvalAltResult`. |
