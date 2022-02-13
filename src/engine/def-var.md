Variable Definition Filter
==========================

{{#include ../links.md}}

Although it is easy to disable variable _[shadowing]_ via [`Engine::set_allow_shadowing`][options],
sometimes more fine-grained control is needed.

For example, it may be the case that not _all_ variables [shadowing] must be disallowed, but that only
a particular variable name needs to be protected and not others.  Or only under very special
circumstances.

Under this scenario, it is possible to provide a _filter_ closure to the [`Engine`] via
`Engine::on_def_var` that traps variable definitions (i.e. [`let`][variable] or [`const`][constant]
statements) in a Rhai script.

The filter is called when a [variable] or [constant] is defined both during runtime and compilation.

```rust,no_run
let mut engine = Engine::new();

// Register a variable definition filter.
engine.on_def_var(|name, is_runtime, is_const, block_level, will_shadow, context| {
    match (name, is_const) {
        // Disallow defining 'MYSTIC_NUMBER' as a constant!
        ("MYSTIC_NUMBER", true) => Ok(false),
        // Disallow defining constants not at global level!
        (_, true) if block_level > 0 => Ok(false),
        // Throw any exception you like...
        ("hello", _) => Err(EvalAltResult::ErrorVariableNotFound(name.to_string(), Position::NONE).into()),
        // Return Ok(true) to continue with normal variable definition.
        _ => Ok(true)
    }
});
```


Function Signature
------------------

The function signature passed to `Engine::on_def_var` takes the following form.

> `Fn(name: &str, is_runtime: bool, is_const: bool, block_level: usize, will_shadow: bool, context: &EvalContext) -> Result<bool, Box<EvalAltResult>>`

where:

| Parameter     |              Type               | Description                                                                                                                                                                                                                                                |
| ------------- | :-----------------------------: | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `name`        |             `&str`              | [variable] name                                                                                                                                                                                                                                            |
| `is_runtime`  |             `bool`              | `true` if the [variable] definition event happens during runtime, `false` if during compilation                                                                                                                                                            |
| `is_const`    |             `bool`              | `true` if the definition is a [`const`][constant]; `false` if it is a [`let`][variable]                                                                                                                                                                    |
| `block_level` |             `usize`             | **compile-time**: the current expression nesting level; the global level is zero, and each sub-expression adds one to the level<br/>**runtime**: the current statement block nesting level; the global level is zero, and each block adds one to the level |
| `will_shadow` |             `bool`              | will this [variable] _[shadow]_ an existing [variable] of the same name?                                                                                                                                                                                   |
| `context`     | [`&EvalContext`][`EvalContext`] | the current _evaluation context_                                                                                                                                                                                                                           |

and [`EvalContext`] is a type that encapsulates the current _evaluation context_.

### Return value

The return value is `Result<bool, Box<EvalAltResult>>` where:

| Value                     | Description                                        |
| ------------------------- | -------------------------------------------------- |
| `Ok(true)`                | normal [variable] definition should continue       |
| `Ok(false)`               | [throws][exception] a runtime or compilation error |
| `Err(Box<EvalAltResult>)` | error that is reflected back to the [`Engine`]     |

During compilation (i.e. when `is_runtime` is `false`), `EvalAltResult::ErrorParsing` is passed
through as the compilation error.  All other errors map to `ParseErrorType::ForbiddenVariable`.
