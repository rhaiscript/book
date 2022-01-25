Call Stack
==========

{{#include ../../links.md}}

The [debugger] keeps a _call stack_ of [function] calls with argument values. This call stack can be
examined to determine the control flow at any particular point.

Each "frame" in the call stack corresponds to one layer of [function] call (script-defined
[functions] only, not Rust native functions), and is represented by the `debugger::CallStackFrame` type.

The call stack, obviously, is not available under [`no_function`].

The `Debugger::call_stack` method returns a slice of all call stack frames.

```rust,no_run
use rhai::debugger::*;

let debugger = &mut context.global_runtime_state_mut().debugger;

// Get depth of the call stack.
let depth = debugger.call_stack().len();

// Display all function calls
for frame in debugger.call_stack().iter() {
    println!("{}", frame);
}
```
