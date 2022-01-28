Debugger State
==============

{{#include ../../links.md}}

Sometimes it is useful to keep a persistent _state_ within the [debugger].

The `Engine::on_debugger` API accepts a function that returns the initial value of the
[debugger's][debugger] state, which is a [`Dynamic`] and can hold any value.

The state can easily be accessed during a [debugger] callback.

```rust,no_run
engine.on_debugger(
    || {
        // Say, use an object map for the debugger state
        let mut state = Map::new();
        // Initialize properties
        state.insert("hello".into(), 42_64.into());
        state.insert("foo".into(), false.into());
        Dynamic::from_map(state)
    },
    |context, node, source, pos| {
        // Get global runtime state
        let global = context.global_runtime_state_mut();

        // Get debugger
        let debugger = &mut global.debugger;

        // Print debugger state - which is an object map
        println!("Current state = {}", debugger.state());

        // Get the state as an object map
        let mut state = debugger.state_mut().write_lock::<Map>().unwrap();

        // Read state
        let hello = state.get("hello").unwrap().as_int().unwrap();

        // Modify state
        state.insert("hello".into(), (hello + 1).into());
        state.insert("foo".into(), true.into());
        state.insert("something_new".into(), "hello, world!".into());

        // Continue with debugging
        Ok(DebuggerCommand::StepInto)
    }
);
```
