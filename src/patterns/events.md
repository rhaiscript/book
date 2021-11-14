Scriptable Event Handler with State
==================================

{{#include ../links.md}}


Usage Scenario
--------------

* A system sends _events_ that must be handled.

* Flexibility in event handling must be provided, through user-side scripting.

* State must be kept between invocations of event handlers.

* Default implementations of event handlers can be provided.


Key Concepts
------------

* An _event handler_ object is declared that holds the following items:
  * [`Engine`] with registered functions serving as an API,
  * [`AST`] of the user script,
  * a [`Scope`] containing state.

* Upon an event, the appropriate event handler function in the script is called via [`Engine::call_fn`][`call_fn`].

* Optionally, trap the `EvalAltResult::ErrorFunctionNotFound` error to provide a default implementation.


Implementation
--------------

### Declare Handler Object

In most cases, it would be simpler to store an [`Engine`] instance together with the handler object
because it only requires registering all API functions only once.

In rare cases where handlers are created and destroyed in a tight loop, a new [`Engine`] instance
can be created for each event. See [_One Engine Instance Per Call_](parallel.md) for more details.

```rust no_run
use rhai::{Engine, Scope, AST, EvalAltResult};

// Event handler
struct Handler {
    // Scripting engine
    pub engine: Engine,
    // Use a custom 'Scope' to keep stored state
    pub scope: Scope<'static>,
    // Program script
    pub ast: AST
}
```

### Register API for Any Custom Type

[Custom types] are often used to hold state. The easiest way to register an entire API is via a [plugin module].

```rust no_run
use rhai::plugin::*;

// A custom type to a hold state value.
#[derive(Debug, Clone, Eq, PartialEq, Hash, Default)]
pub struct SomeType {
    data: i64
}

#[export_module]
mod SomeTypeAPI {
    #[rhai_fn(global)]
    pub fn new_state(value: i64) -> SomeType { ... }
    #[rhai_fn(global)]
    pub fn func1(obj: &mut SomeType) -> bool { ... }
    #[rhai_fn(global)]
    pub fn func2(obj: &mut SomeType) -> bool { ... }
    pub fn process(data: i64) -> i64 { ... }
    #[rhai_fn(get = "value", pure)]
    pub fn get_value(obj: &mut SomeType) -> i64 { obj.data }
    #[rhai_fn(set = "value")]
    pub fn set_value(obj: &mut SomeType, value: i64) { obj.data = value; }
}
```

### Initialize Handler Object

Steps to initialize the event handler:

1. Register an API with the [`Engine`],
2. Create a custom [`Scope`] to serve as the stored state,
3. Add default state variables into the custom [`Scope`] and/or create them by calling an initiation function
4. Get the handler script and [compile][`AST`] it,
5. Store the compiled [`AST`] for future evaluations,
6. Run the [`AST`] to initialize event handler state variables.

```rust no_run
impl Handler {
    pub fn new(path: impl Into<PathBuf>) -> Self {
        let mut engine = Engine::new();

        // Register custom types and API's
        engine.register_type_with_name::<SomeType>("SomeType")
              .register_global_module(exported_module!(SomeTypeAPI).into());

        // Create a custom 'Scope' to hold state
        let mut scope = Scope::new();

        // Add initialized state into the custom 'Scope'
        scope.push("bool_state", false);
        scope.push("some_type_state", SomeType::new(42));

        // Compile the handler script.
        // In a real application you'd be handling errors...
        let ast = engine.compile_file(path).unwrap();

        // Evaluate the script to initialize it and other state variables.
        // In a real application you'd again be handling errors...
        engine.run_ast_with_scope(&mut scope, &ast).unwrap();

        // The event handler is essentially these three items:
        Handler { engine, scope, ast }
    }
}
```

### Hook up events

There is usually an interface or trait that gets called when an event comes from the system.

Mapping an event from the system into a scripted handler is straight-forward:

```rust no_run
impl Handler {
    // Create a new 'Handler'.
    // In a real application you'd be handling errors...
    pub fn new(script: &str) -> Self {
        let engine = Engine::new();
        let mut scope = Scope::new();
        let ast = engine.compile(script).unwrap();

        // Run the 'init' function to initialize the state
        engine.call_fn(&mut scope, &ast, "init", ()).unwrap();

        Self { engine, scope, ast }
    }

    // Say there are three events: 'start', 'end', 'update'.
    // In a real application you'd be handling errors...
    pub fn on_event(&mut self, event_name: &str, event_data: i64) -> Dynamic {
        let engine = &self.engine;
        let scope = &mut self.scope;
        let ast = &self.ast;

        match event_name {
            // The 'start' event maps to function 'start'.
            // In a real application you'd be handling errors...
            "start" => engine.call_fn(scope, ast, "start", (event_data,)).unwrap(),

            // The 'end' event maps to function 'end'.
            // In a real application you'd be handling errors...
            "end" => engine.call_fn(scope, ast, "end", (event_data,)).unwrap(),

            // The 'update' event maps to function 'update'.
            // This event provides a default implementation when the scripted function is not found.
            "update" =>
                engine.call_fn(scope, ast, "update", (event_data,))
                      .or_else(|err| match *err {
                         EvalAltResult::ErrorFunctionNotFound(fn_name, _) if fn_name == "update" => {
                            // Default implementation of 'update' event handler
                            self.scope.set_value("some_type_state", SomeType::new(42));
                            // Turn function-not-found into a success
                            Ok(Dynamic::UNIT)
                         }
                         _ => Err(err)
                      }).unwrap(),

            // In a real application you'd be handling unknown events...
            _ => panic!("unknown event: {}", event_name)
        }
    }
}
```

### Sample Handler Script

Because the stored state is kept in a custom [`Scope`], it is possible for all functions defined
in the handler script to access and modify these state variables.

The API registered with the [`Engine`] can be also used throughout the script.

```rust no_run
// Initialize state
fn init() {
    let bool_state = false;
    let some_type_state = new_state(42);
}

// 'start' event handler
fn start(data) {
    if bool_state {
        throw "Already started!";
    }
    if some_type_state.func1() || some_type_state.func2() {
        throw "Conditions not yet ready to start!";
    }
    bool_state = true;
    some_type_state.value = data;
}

// 'end' event handler
fn end(data) {
    if !bool_state {
        throw "Not yet started!";
    }
    if some_type_state.func1() || some_type_state.func2() {
        throw "Conditions not yet ready to start!";
    }
    bool_state = false;
    some_type_state.value = data;
}

// 'update' event handler
fn update(data) {
    some_type_state.value += process(data);
}
```
