Register with the Debugger
==========================

{{#include ../../links.md}}

Hooking up a debugging interface is as simple as providing closures to the [`Engine`]'s built-in
debugger via `Engine::register_debugger`.

```rust
use rhai::debugger::{ASTNode, DebuggerCommand};

let mut engine = Engine::new();

engine.register_debugger(
    // Provide a callback to initialize the debugger state
    |engine| { ... },
    // Provide a callback for each debugging step
    |context, event, node, source, pos| {
        ...

        DebuggerCommand::StepOver
    }
);
```

~~~admonish tip.small "Tip: Accessing the `Debugger`"

The type `debugger::Debugger` allows for manipulating [break-points], among others.

The [`Engine`]'s debugger instance can be accessed via `context.global_runtime_state().debugger()` (immutable)
or `context.global_runtime_state_mut().debugger_mut()` (mutable).
~~~


Callback Functions Signature
----------------------------

There are two callback functions to register for the debugger.

The first is simply a function to initialize the state of the debugger (a [`Dynamic`] value),
with the following signature.

> `Fn() -> Dynamic`

The second callback is a function which will be called by the debugger during each step, with the
following signature.

> `Fn(context: EvalContext, event: debugger::DebuggerEvent, node: ASTNode, source: &str, pos: Position) -> Result<debugger::DebuggerCommand, Box<EvalAltResult>>`

where:

| Parameter |      Type       | Description                                                                                                                  |
| --------- | :-------------: | ---------------------------------------------------------------------------------------------------------------------------- |
| `context` | [`EvalContext`] | the current _evaluation context_                                                                                             |
| `event`   | `DebuggerEvent` | an `enum` indicating the event that triggered the debugger                                                                   |
| `node`    |    `ASTNode`    | an `enum` with two variants: `Expr` or `Stmt`, corresponding to the current expression node or statement node in the [`AST`] |
| `source`  |     `&str`      | the source of the current [`AST`], or empty if none                                                                          |
| `pos`     |   `Position`    | position of the current node, same as `node.position()`                                                                      |

and [`EvalContext`] is a type that encapsulates the current _evaluation context_.

### Event

The `event` parameter of the second closure passed to `Engine::register_debugger` contains a
`debugger::DebuggerEvent` which is an `enum` with the following variants.

| `DebuggerEvent` variant          | Description                                                                                     |
| -------------------------------- | ----------------------------------------------------------------------------------------------- |
| `Start`                          | the debugger is triggered at the beginning of evaluation                                        |
| `Step`                           | the debugger is triggered at the next step of evaluation                                        |
| `BreakPoint(`_n_`)`              | the debugger is triggered by the _n_-th [break-point]                                           |
| `FunctionExitWithValue(`_r_`)`   | the debugger is triggered by a function call returning with value _r_ which is `&Dynamic`       |
| `FunctionExitWithError(`_err_`)` | the debugger is triggered by a function call exiting with error _err_ which is `&EvalAltResult` |
| `End`                            | the debugger is triggered at the end of evaluation                                              |

### Return value

```admonish tip.side.wide "Tip: Initialization"

When a script starts evaluation, the debugger always stops at the very _first_ [`AST`] node
with the `event` parameter set to `DebuggerStatus::Start`.

This allows initialization to be done (e.g. setting up [break-points]).
```

The second closure passed to `Engine::register_debugger` will be called when stepping into or over
expressions and statements, or when [break-points] are hit.

The return type of the closure is `Result<debugger::DebuggerCommand, Box<EvalAltResult>>`.

If an error is returned, the script evaluation at that particular instance returns with that
particular error. It is thus possible to _abort_ the script evaluation by returning an error that is
not _catchable_, such as `EvalAltResult::ErrorTerminated`.

If no error is returned, then the return `debugger::DebuggerCommand` variant determines the
continued behavior of the debugger.

| `DebuggerCommand` variant | Behavior                                                                                                                        | `gdb` equivalent |
| ------------------------- | ------------------------------------------------------------------------------------------------------------------------------- | :--------------: |
| `Continue`                | continue with normal script evaluation                                                                                          |    `continue`    |
| `StepInto`                | run to the next expression or statement, diving into functions                                                                  |      `step`      |
| `StepOver`                | run to the next expression or statement, skipping over functions                                                                |                  |
| `Next`                    | run to the next statement, skipping over functions                                                                              |      `next`      |
| `FunctionExit`            | run to the end of the current function call; debugger is triggered _before_ the function call returns and the [`Scope`] cleared |     `finish`     |
