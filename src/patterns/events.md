Scriptable Event Handler with State
==================================

{{#include ../links.md}}


Usage Scenario
--------------

* A system sends _events_ that must be handled.

* Flexibility in event handling must be provided, through user-side scripting.

* State must be kept between invocations of event handlers.

* State may be provided by the system or the user, or both.

* Default implementations of event handlers can be provided.


Key Concepts
------------

* An _event handler_ object is declared that holds the following items:
  * [`Engine`] with registered functions serving as an API,
  * [`AST`] of the user script,
  * a [`Scope`] containing system-provided default state.

* User-provided state is initialized by a [function] called via [`Engine::call_fn_raw`][`call_fn`].

* Upon an event, the appropriate event handler [function] in the script is called via [`Engine::call_fn`][`call_fn`].

* Optionally, trap the `EvalAltResult::ErrorFunctionNotFound` error to provide a default implementation.


Implementation
--------------

### Declare handler object

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

### Register API for custom types

[Custom types] are often used to hold state. The easiest way to register an entire API is via a [plugin module].

```rust no_run
use rhai::plugin::*;

// A custom type to a hold state value.
#[derive(Debug, Clone, Eq, PartialEq, Hash, Default)]
pub struct TestStruct {
    data: i64
}

// Plugin module containing API to TestStruct
#[export_module]
mod test_struct_api {
    #[rhai_fn(global)]
    pub fn new_state(value: i64) -> TestStruct {
        TestStruct { data: value }
    }
    #[rhai_fn(global)]
    pub fn func1(obj: &mut TestStruct) -> bool {
                :
                :
    }
    #[rhai_fn(global)]
    pub fn func2(obj: &mut TestStruct) -> i64 {
                :
                :
    }
    pub fn process(data: i64) -> i64 {
                :
                :
    }
    #[rhai_fn(get = "value", pure)]
    pub fn get_value(obj: &mut TestStruct) -> i64 {
        obj.data
    }
    #[rhai_fn(set = "value")]
    pub fn set_value(obj: &mut TestStruct, value: i64) {
        obj.data = value;
    }
}
```

### Initialize handler object

Steps to initialize the event handler:

1. Register an API with the [`Engine`],
2. Create a custom [`Scope`] to serve as the stored state,
3. Add default state variables into the custom [`Scope`],
4. Optionally, call an initiation [function] to create new state variables; `Engine::call_fn_raw` is
   used instead of `Engine::call_fn` so that variables created inside the [function] will not be
   removed from the custom [`Scope`] upon exit,
5. Get the handler script and [compile][`AST`] it,
6. Store the compiled [`AST`] for future evaluations,
7. Run the [`AST`] to initialize event handler state variables.

```rust no_run
impl Handler {
    // Create a new 'Handler'.
    pub fn new(path: impl Into<PathBuf>) -> Self {
        let mut engine = Engine::new();

        // Register custom types and API's
        engine.register_type_with_name::<TestStruct>("TestStruct")
              .register_global_module(exported_module!(test_struct_api).into());

        // Create a custom 'Scope' to hold state
        let mut scope = Scope::new();

        // Add any system-provided state into the custom 'Scope'.
        // Constants can be used to optimize the script.
        scope.push_constant("MY_CONSTANT", 42_i64);

        // Compile the handler script.
        // In a real application you'd be handling errors...
        let ast = engine.compile_file_with_scope(&mut scope, path).unwrap();

        // Run the 'init' function to initialize the state, retaining variables.
        // Use 'call_fn_raw' instead of 'call_fn' in order to retain new variables.
        // In a real application you'd again be handling errors...
        engine.call_fn_raw(&mut scope, &ast, false, false, "init", None, []).unwrap();
        //                                          ^^^^^ do not rewind scope

        // The event handler is essentially these three items:
        Self { engine, scope, ast }
    }
}
```

### Hook up events

There is usually an interface or trait that gets called when an event comes from the system.

Mapping an event from the system into a scripted handler is straight-forward:

```rust no_run
impl Handler {
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
                            self.scope.set_value("obj_state", TestStruct::new(42));
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

### Sample handler script

Because the stored state is kept in a custom [`Scope`], it is possible for all [functions] defined
in the handler script to access and modify these state variables.

The API registered with the [`Engine`] can be also used throughout the script.

```rust no_run
// Initialize user-provided state (overrides system-provided state, if any).
// When 'call_fn_raw' is used to call this, the scope is not rewound and
// any new variables introduced will stay inside the custom 'Scope'.
fn init() {
    // Add 'bool_state' and 'obj_state' as new state variables
    let bool_state = false;
    let obj_state = new_state(0);
}

// 'start' event handler
fn start(data) {
    if bool_state {
        throw "Already started!";
    }
    if obj_state.func1() || obj_state.func2() {
        throw "Conditions not yet ready to start!";
    }
    bool_state = true;
    // Constant 'MY_CONSTANT' in custom scope is also visible
    obj_state.value = data + MY_CONSTANT;
}

// 'end' event handler
fn end(data) {
    if !bool_state {
        throw "Not yet started!";
    }
    if obj_state.func1() || obj_state.func2() {
        throw "Conditions not yet ready to start!";
    }
    bool_state = false;
    obj_state.value = data;
}

// 'update' event handler
fn update(data) {
    obj_state.value += process(data);
}
```


Alternate Style
---------------

|                                             |                                                 Main style                                                  |                       Alternate style                       |
| ------------------------------------------- | :---------------------------------------------------------------------------------------------------------: | :---------------------------------------------------------: |
| States store                                |                                              custom [`Scope`]                                               |                        [object map]                         |
| Access state variable                       |                                              normal [variable]                                              |          [property][object map] of `this` pointer           |
| Local variable may _shadow_ state variable? |                                                     yes                                                     |                             no                              |
| Add new state variable                      |                                           `init` [function] only                                            |                       all [functions]                       |
| Benefits                                    |                                                   simple                                                    |                          versatile                          |
| Disadvantages                               | &bull; cannot add new variables in [functions] (apart from `init`)<br/>&bull; easy variable name collisions | `this.xxx` all over the place (looks a lot like JavaScript) |

### Disadvantages of the main style

In the style above, new user state variables are introduced by evaluating a special initialization
script [function] (e.g. `init`) that defines them, and `Engine::call_fn_raw` is used to keep them
inside the custom [`Scope`].

However, this style has the disadvantage that no other [function] can introduce new state variables,
otherwise they'd simply _shadow_ existing variables in the custom [`Scope`].  Thus, [functions] are
called during events via `Engine::call_fn` which does not retain any variables.

When there are a large number of state variables, this style also makes it easy for local variables
defined in user functions to accidentally _shadow_ a state variable with a variable that just
happens to be the same name.

```rust no_run
// 'start' event handler
fn start(data) {
    let bool_state = false; // <- oops! bad variable name!
        
        :                   // there is now no way to access the
        :                   // state variable 'bool_state'...

    if bool_state {         // <- 'bool_state' is not the right thing
        ...                 //    unless this is what you actually want
    }

        :
        :
}
```

### Keep state in object map

Another style that allows new user state variables is to package them all inside an [object map],
which is then exposed via the `this` pointer.  State variables can be freely created inside the
[object map] by _all_ [functions] (not just the initialization [function]).

```rust no_run
use rhai::{Engine, Scope, Map, AST, EvalAltResult};

// Event handler
struct Handler {
    // Scripting engine
    pub engine: Engine,
    // Use an object map to keep stored state
    pub states: Map,
    // Program script
    pub ast: AST
}
```

### Bind object map to `this` pointer

Initialization can simply be done via binding the `this` pointer.

```rust no_run
// Use an object map to hold state
let mut states: Dynamic = Map::new().into();

// Default states can be added
states.insert("bool_state".into(), Dynamic::FALSE);

// The custom 'Scope' is no longer used to hold state
let mut scope = Scope::new();

// But variables/constants can still be added
scope.push_constant("MY_CONSTANT", 42_i64);

// Use 'call_fn_raw' instead of 'call_fn' to bind the 'this' pointer
// In a real application you'd again be handling errors...
engine.call_fn_raw(&mut scope, &ast, false, true, "init", Some(&mut states), []).unwrap();
//                                          ^^^^          ^^^^^^^^^^^^^^^^^
//                                      rewind scope      bind 'this' pointer
```

### Bind `this` pointer during events handling

Events handling should also use `Engine::call_fn_raw` to bind the [object map] containing global
states to the `this` pointer.

```rust no_run
pub fn on_event(&mut self, event_name: &str, event_data: i64) -> Dynamic {
    let engine = &self.engine;
    let states = &mut self.states;
    let ast = &self.ast;

    match event_name {
        // In a real application you'd be handling errors...
        "start" => engine.call_fn_raw(&mut Scope::new(), ast, false, true, "start",
                                      Some(states), [event_data.into()]).unwrap(),
                                   // ^^^^^^^^^^^^ bind 'this' pointer
            :
            :
    }
}
```

### Handler script

Because the stored state is kept in an [object map], which in turn is mapped to `this`, it is
necessary for [functions] to access and modify these state variables via the `this` pointer.

```rust no_run
// Initialize user-provided state (overrides system-provided state, if any).
// State is stored inside an object map bound to 'this'.
fn init() {
    // Add 'bool_state' as new state variable if one does not exist
    if !("bool_state" in this) {
        this.bool_state = false;
    }
    // Add 'obj_state' as new state variable (overwrites any existing)
    this.obj_state = new_state(0);
}

// 'start' event handler
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
    this.obj_state.value = data + MY_CONSTANT;
}

// 'end' event handler
fn end(data) {
    if !this.bool_state || !("start_mode" in this) {
        throw "Not yet started!";
    }
    if this.obj_state.func1() || this.obj_state.func2() {
        throw "Conditions not yet ready to start!";
    }
    this.bool_state = false;
    this.obj_state.value = data;
}

// 'update' event handler
fn update(data) {
    this.obj_state.value += process(data);
}
```
