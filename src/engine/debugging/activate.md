Activate Debugger
=================

{{#include ../../links.md}}

Hooking up a custom [debugger] and activating it is as simple as providing a closure to the
[`Engine`] via `Engine::on_debugger`.

```rust,no_run
use rhai::debugger::*;

let mut engine = Engine::new();

engine.on_debugger(|context, node, source, pos| {
    ...

    DebuggerCommand::StepOver
});
```


Callback Function Signature
---------------------------

The signature of a [debugger] callback is as follows.

> `Fn(context: &mut EvalContext, node: ASTNode, source: &str, pos: Position) -> debugger::DebuggerCommand`

where:

| Parameter |                Type                 | Description                                                                                                                  |
| --------- | :---------------------------------: | ---------------------------------------------------------------------------------------------------------------------------- |
| `context` | [`&mut EvalContext`][`EvalContext`] | mutable reference to the current _evaluation context_                                                                        |
| `node`    |         [`ASTNode`][`AST`]          | an `enum` with two variants: `Expr` or `Stmt`, corresponding to the current expression node or statement node in the [`AST`] |
| `source`  |               `&str`                | the source of the current [`AST`], or empty if none                                                                          |
| `pos`     |             `Position`              | position of the current node, same as `node.position()`                                                                      |

and [`EvalContext`] is a type that encapsulates the current _evaluation context_.

### Return value

The closure passed to `Engine::on_debugger` will be called when stepping into or over expressions
and statements, or when [break-points] are hit.

The return type of the closure provided to `Engine::on_debugger` is `debugger::DebuggerCommand`.

|   Return value of closure   | Behavior                                                        | `gdb` equivalent |
| :-------------------------: | --------------------------------------------------------------- | :--------------: |
| `DebuggerCommand::Continue` | continue with normal script evaluation                          |    `continue`    |
| `DebuggerCommand::StepInto` | stop at the next expression or statement, diving into functions |      `step`      |
| `DebuggerCommand::StepOver` | stop at the next statement, skipping over functions             |      `next`      |


Debugger
--------

`EvalContext::global_runtime_state_mut().debugger` (of type `debugger::Debugger`) allows access to
the [`Engine`]'s [debugger], which allows manipulating [break-points], among others.
