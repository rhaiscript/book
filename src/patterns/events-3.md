Scriptable Event Handler with State<br/>Map Style
=================================================

{{#include ../links.md}}


```admonish example

A runnable example of this implementation is included.

See the [_Examples_]({{rootUrl}}/start/examples/rust.md) section for details.
```


I Hate `this`!  How Can I Get Rid of It?
----------------------------------------

You're using Rust and you don't want people to think you're writing lowly JavaScript?

Taking inspiration from the [_JS Style_](events-2.md), a slight modification of the
[_Main Style_](events-1.md) is to store all states inside an [object map] inside a custom [`Scope`].

Nevertheless, instead of writing `this.variable_name` everywhere to access a state variable
(in the [_JS Style_](events-2.md)), you'd write `state.variable_name` instead.

It is up to you to decide whether this is an improvement!


Handler Initialization
----------------------

```admonish note.side "No shadowing 'state'"

Notice that a [variable definition filter] is used to prevent [shadowing] of the states [object map].
```

Implementation wise, this style follows closely the [_Main Style_](events-1.md), but a single
[object map] is added to the custom [`Scope`] which holds all state values.

Global [constants] can still be added to the custom [`Scope`] as normal and used through the script.

Calls to the `init` [function] no longer need to avoid rewinding the [`Scope`] because state
[variables] are added as properties under the states [object map].

```rust
impl Handler {
    // Create a new 'Handler'.
    pub fn new(path: impl Into<PathBuf>) -> Self {
        let mut engine = Engine::new();

        // Forbid shadowing of 'state' variable
        engine.on_def_var(|_, info, _| Ok(info.name != "state"));

                    :
            // Code omitted
                    :

        // Use an object map to hold state
        let mut states = Map::new();

        // Default states can be added
        states.insert("bool_state".into(), Dynamic::FALSE);

        // Add the main states-holding object map and call it 'state'
        scope.push("state", states);

        // Just a simple 'call_fn' can do here because we're rewinding the 'Scope'
        // In a real application you'd again be handling errors...
        engine.call_fn(&mut scope, &ast, "init", ()).unwrap();

                    :
            // Code omitted
                    :

        Self { engine, scope, ast }
    }
}
```


Handler Scripting Style
-----------------------

The stored state is kept in an [object map] in the custom [`Scope`].

In this example, that [object map] is named `state`, but it can be any name.

### User-defined functions in state

Because an [object map] is used to hold state values, it is even possible to add user-defined
[functions], leveraging the [OOP] support for [object maps].

However, within these user-defined [functions], the `this` pointer binds to the [object map].
Therefore, the variable-accessing syntax is different from the main body of the script.

```js
fn do_action() {
    // Access state: `state.xxx`
    state.number = 42;

    // Add OOP functions - you still need to use `this`...
    state.log = |x| print(`State = ${this.value}, data = ${x}`);
}
```

### Sample script

```js
/// Initialize user-provided state.
/// State is stored inside an object map bound to 'state'.
fn init() {
    // Add 'bool_state' as new state variable if one does not exist
    if !("bool_state" in state) {
        state.bool_state = false;
    }
    // Add 'obj_state' as new state variable (overwrites any existing)
    state.obj_state = new_state(0);

    // Can also add OOP-style functions!
    state.log = |x| print(`State = ${this.obj_state.value}, data = ${x}`);
}

/// 'start' event handler
fn start(data) {
    // Can detect system-provided default states!
    // Access state variables in 'state'
    if state.bool_state {
        throw "Already started!";
    }

    // New values can be added to the state
    state.start_mode = data;

    if state.obj_state.func1() || state.obj_state.func2() {
        throw "Conditions not yet ready to start!";
    }
    state.bool_state = true;
    state.obj_state.value = data;

    // Constant 'MY_CONSTANT' in custom scope is also visible!
    print(`MY_CONSTANT = ${MY_CONSTANT}`);
}

/// 'end' event handler
fn end(data) {
    if !state.bool_state || !("start_mode" in state) {
        throw "Not yet started!";
    }
    if !state.obj_state.func1() && !state.obj_state.func2() {
        throw "Conditions not yet ready to end!";
    }
    state.bool_state = false;
    state.obj_state.value = data;
}

/// 'update' event handler
fn update(data) {
    state.obj_state.value += process(data);

    // Call user-defined function OOP-style!
    state.log(data);
}
```
