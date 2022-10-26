Call Stack
==========

{{#include ../../links.md}}

```admonish info.side.wide "Call stack frames"

Each "frame" in the call stack corresponds to one layer of [function] call (script-defined
or native Rust).

A call stack frame has the type `debugger::CallStackFrame`.
```

The [debugger] keeps a _call stack_ of [function] calls with argument values.

This call stack can be examined to determine the control flow at any particular point.

The `Debugger::call_stack` method returns a slice of all call stack frames.

```rust
use rhai::debugger::*;

let debugger = &mut context.global_runtime_state_mut().debugger;

// Get depth of the call stack.
let depth = debugger.call_stack().len();

// Display all function calls
for frame in debugger.call_stack().iter() {
    println!("{frame}");
}
```
