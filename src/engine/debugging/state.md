Debugger State
==============

{{#include ../../links.md}}

Sometimes it is useful to keep a persistent _state_ within the [debugger].

The `Engine::register_debugger` API accepts a function that returns the initial value of the
[debugger's][debugger] state, which is a [`Dynamic`] and can hold any value.

This state value is the stored into the [debugger]'s custom state.


Access the Debugger State
-------------------------

Use `EvalContext::global_runtime_state_mut().debugger` to gain access to the current
[`debugger::Debugger`] instance.

The following [`debugger::Debugger`] methods allow access to the custom [debugger] state.

| Method      |          Parameter type           |         Return type         | Description                                     |
| ----------- | :-------------------------------: | :-------------------------: | ----------------------------------------------- |
| `state`     |              _none_               |   [`&Dynamic`][`Dynamic`]   | returns the custom state                        |
| `state_mut` |              _none_               | [`&mut Dynamic`][`Dynamic`] | returns a mutable reference to the custom state |
| `set_state` | [`impl Into<Dynamic>`][`Dynamic`] |           _none_            | sets the value of the custom state              |


Example
-------

```rust
engine.register_debugger(
    |engine| {
        // Say, use an object map for the debugger state
        let mut state = Map::new();
        // Initialize properties
        state.insert("hello".into(), 42_64.into());
        state.insert("foo".into(), false.into());
        Dynamic::from_map(state)
    },
    |context, node, source, pos| {
        // Print debugger state - which is an object map
        let state = context.global_runtime_state_mut().debugger.state();
        println!("Current state = {state}");

        // Get the state as an object map
        let mut state = context.global_runtime_state_mut()
                               .debugger.state_mut()
                               .write_lock::<Map>().unwrap();

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
