Scriptable Event Handler with State<br/>JS Style
================================================

{{#include ../links.md}}


```admonish example

A runnable example of this implementation is included.

See the [_Examples_]({{rootUrl}}/start/examples/rust.md) section for details.
```


Keep State in Object Map
------------------------

This style allows defining new user state [variables] everywhere by packaging them all inside an
[object map], which is then exposed via the `this` pointer.

Because this scripting style resembles JavaScript, it is so named.

State variables can be freely created by _all_ [functions] (not just the `init` [function]).

The event handler type needs to hold this [object map] instead of a custom [`Scope`].

```rust
use rhai::{Engine, Scope, Dynamic, AST};

// Event handler
struct Handler {
    // Scripting engine
    pub engine: Engine,
    // The custom 'Scope' can be used to hold global constants
    pub scope: Scope<'static>,
    // Use an object map (as a 'Dynamic') to keep stored state
    pub states: Dynamic,
    // Program script
    pub ast: AST
}
```


Bind Object Map to `this` Pointer
---------------------------------

Initialization can simply be done via binding the [object map] containing global states to the
`this` pointer.

```rust
impl Handler {
    // Create a new 'Handler'.
    pub fn new(path: impl Into<PathBuf>) -> Self {
        let mut engine = Engine::new();

                    :
            // Code omitted
                    :

        // Use an object map to hold state
        let mut states = Map::new();

        // Default states can be added
        states.insert("bool_state".into(), Dynamic::FALSE);

        // Convert the object map into 'Dynamic'
        let mut states: Dynamic = states.into();

        // Use 'call_fn_with_options' instead of 'call_fn' to bind the 'this' pointer
        let options = CallFnOptions::new()
                        .eval_ast(false)                // do not re-evaluate the AST
                        .rewind_scope(true)             // rewind scope
                        .bind_this_ptr(&mut states);    // bind the 'this' pointer

        // In a real application you'd again be handling errors...
        engine.call_fn_with_options(options, &mut scope, &ast, "init", ()).unwrap();

                    :
            // Code omitted
                    :

        Self { engine, scope, states, ast }
    }
}
```


Bind `this` Pointer During Events Handling
------------------------------------------

Events handling should also use `Engine::call_fn_with_options` to bind the [object map] containing
global states to the `this` pointer via `CallFnOptions::this_ptr`.

```rust
pub fn on_event(&mut self, event_name: &str, event_data: i64) -> Dynamic {
    let engine = &self.engine;
    let scope = &mut self.scope;
    let states = &mut self.states;
    let ast = &self.ast;

    let options = CallFnOptions::new()
                    .eval_ast(false)                // do not re-evaluate the AST
                    .rewind_scope(true)             // rewind scope
                    .bind_this_ptr(&mut states);    // bind the 'this' pointer

    match event_name {
        // In a real application you'd be handling errors...
        "start" => engine.call_fn_with_options(options, scope, ast, "start", (event_data,)).unwrap(),
            :
            :
    }
}
```


Handler Scripting Style
-----------------------

```admonish note.side "No shadowing"

Notice that `this` can never be [shadowed][shadowing] because it is not a valid [variable] name.
```

Because the stored state is kept in an [object map], which in turn is bound to `this`, it is
necessary for [functions] to always access or modify these state [variables] via the `this` pointer.

As it is impossible to declare a local [variable] named `this`, there is no risk of accidentally
_[shadowing]_ a state [variable].

Because an [object map] is used to hold state values, it is even possible to add user-defined
[functions], leveraging the [OOP] support for [object maps].

### Sample script

```js
/// Initialize user-provided state.
/// State is stored inside an object map bound to 'this'.
fn init() {
    // Can detect system-provided default states!
    // Add 'bool_state' as new state variable if one does not exist
    if !("bool_state" in this) {
        this.bool_state = false;
    }
    // Add 'obj_state' as new state variable (overwrites any existing)
    this.obj_state = new_state(0);

    // Can also add OOP-style functions!
    this.log = |x| print(`State = ${this.obj_state.value}, data = ${x}`);
}

/// 'start' event handler
fn start(data) {
    // Access state variables via 'this'
    if this.bool_state {
        throw "Already started!";
    }

    // New state variables can be created anywhere
    this.start_mode = data;

    if this.obj_state.func1() || this.obj_state.func2() {
        throw "Conditions not yet ready to start!";
    }
    this.bool_state = true;
    this.obj_state.value = data;

    // Constant 'MY_CONSTANT' in custom scope is also visible!
    print(`MY_CONSTANT = ${MY_CONSTANT}`);
}

/// 'end' event handler
fn end(data) {
    if !this.bool_state || !("start_mode" in this) {
        throw "Not yet started!";
    }
    if !this.obj_state.func1() && !this.obj_state.func2() {
        throw "Conditions not yet ready to end!";
    }
    this.bool_state = false;
    this.obj_state.value = data;
}

/// 'update' event handler
fn update(data) {
    this.obj_state.value += process(data);

    // Call user-defined function OOP-style!
    this.log(data);
}
```
