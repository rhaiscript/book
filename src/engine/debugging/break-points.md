Break-Points
============

{{#include ../../links.md}}

A _break-point_ **always** stops the current evaluation and calls the [debugging][debugger]
callback.

A break-point is represented by the `debugger::BreakPoint` type, which is an `enum` with
three variants.

| `BreakPoint` variant                     | Description                                                                                                                                 |
| ---------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------- |
| `AtPosition { source, pos, enabled }`    | breaks at the specified position in the specified source (empty if none);<br/>if `pos` is at beginning of line, breaks anywhere on the line |
| `AtFunctionName { name, enabled }`       | breaks when a function matching the specified name is called (can be [operator])                                                            |
| `AtFunctionCall { name, args, enabled }` | breaks when a function matching the specified name (can be [operator]) and the specified number of arguments is called                      |
| `AtProperty { name, enabled }`           | breaks at the specified property access                                                                                                     |

The following [`debugger::Debugger`] methods allow access of break-points for manipulation.

| Method             |      Return type       | Description                                       |
| ------------------ | :--------------------: | ------------------------------------------------- |
| `break_points`     |    `&[BreakPoint]`     | returns a slice of all `BreakPoint`'s             |
| `break_points_mut` | `&mut Vec<BreakPoint>` | returns a mutable reference to all `BreakPoint`'s |

```rust,no_run
use rhai::debugger::*;

let debugger = &mut context.global_runtime_state_mut().debugger;

// Get number of break-points.
let num_break_points = debugger.break_points().len();

// Add a new break-point on calls to 'foo(_, _, _)'
debugger.break_points_mut().push(
    BreakPoint::AtFunctionCall { name: "foo".into(), args: 3 }
);

// Display all break-points
for bp in debugger.break_points().iter() {
    println!("{}", bp);
}

// Clear all break-points
debugger.break_points_mut().clear();
```
