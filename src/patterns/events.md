Scriptable Event Handler with State
==================================

{{#include ../links.md}}


```admonish info "Usage scenario"

* A system sends _events_ that must be handled.

* Flexibility in event handling must be provided, through user-side scripting.

* State must be kept between invocations of event handlers.

* State may be provided by the system or the user, or both.

* Default implementations of event handlers can be provided.
```

```admonish abstract "Key concepts"

* An _event handler_ object is declared that holds the following items:
  * [`Engine`] with registered functions serving as an API,
  * [`AST`] of the user script,
  * [`Scope`] containing system-provided default state.

* User-provided state is initialized by a [function] called via [`Engine::call_fn_raw`][`call_fn`].

* Upon an event, the appropriate event handler [function] in the script is called via
  [`Engine::call_fn`][`call_fn`].

* Optionally, trap the `EvalAltResult::ErrorFunctionNotFound` error to provide a default implementation.
```


Basic Infrastructure
--------------------

### Declare handler object

In most cases, it would be simpler to store an [`Engine`] instance together with the handler object
because it only requires registering all API functions only once.

In rare cases where handlers are created and destroyed in a tight loop, a new [`Engine`] instance
can be created for each event. See [_One Engine Instance Per Call_](parallel.md) for more details.

```rust,no_run
use rhai::{Engine, Scope, AST};

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

```rust,no_run
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
    }
    #[rhai_fn(global)]
    pub fn func2(obj: &mut TestStruct) -> i64 {
                :
    }
    pub fn process(data: i64) -> i64 {
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
3. Add default state [variables] into the custom [`Scope`],
4. Optionally, call an initiation [function] to create new state [variables]; `Engine::call_fn_raw`
   is used instead of `Engine::call_fn` so that [variables] created inside the [function] will not
   be removed from the custom [`Scope`] upon exit,
5. Get the handler script and [compile][`AST`] it,
6. Store the compiled [`AST`] for future evaluations,
7. Run the [`AST`] to initialize event handler state [variables].

```rust,no_run
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

                    :
                    :
        // Initialize state variables
                    :
                    :

        // Compile the handler script.
        // In a real application you'd be handling errors...
        let ast = engine.compile_file_with_scope(&mut scope, path).unwrap();

        // The event handler is essentially these three items:
        Self { engine, scope, ast }
    }
}
```

### Hook up events

There is usually an interface or trait that gets called when an event comes from the system.

Mapping an event from the system into a scripted handler is straight-forward, via `Engine::call_fn`.

```rust,no_run
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
                          EvalAltResult::ErrorFunctionNotFound(fn_name, _)
                              if fn_name.starts_with("update") =>
                          {
                              // Default implementation of 'update' event handler.
                              self.scope.set_value("obj_state", TestStruct::new(42));
                              // Turn function-not-found into a success.
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


Scripting Styles
----------------

Depending on needs and scripting style, there are three different ways to implement this pattern.

|                                                   |                                          [Main style](events-1.md)                                          |                               [JS style](events-2.md)                               |                           [Map style](events-3.md)                           |
| ------------------------------------------------- | :---------------------------------------------------------------------------------------------------------: | :---------------------------------------------------------------------------------: | :--------------------------------------------------------------------------: |
| States store                                      |                                              custom [`Scope`]                                               |                            [object map] bound to `this`                             |                       [object map] in custom [`Scope`]                       |
| Access state [variable]                           |                                              normal [variable]                                              |                          [property][object map] of `this`                           |                   [property][object map] of state variable                   |
| Access global [constants]?                        |                                                     yes                                                     |                                         yes                                         |                                     yes                                      |
| Add new state [variable]?                         |                                           `init` [function] only                                            |                                   all [functions]                                   |                               all [functions]                                |
| Add new global [constants]?                       |                                                     yes                                                     |                                         no                                          |                                      no                                      |
| [OOP]-style [functions] on states?                |                                                     no                                                      |                                         yes                                         |                                     yes                                      |
| Detect system-provided initial states?            |                                                     no                                                      |                                         yes                                         |                                     yes                                      |
| Local [variable] may _[shadow]_ state [variable]? |                                                     yes                                                     |                                         no                                          |                                     yes                                      |
| Benefits                                          |                                                   simple                                                    |                                   fewer surprises                                   |                                  versatile                                   |
| Disadvantages                                     | <ul><li>no new variables in [functions] (apart from `init`)</li><li>easy variable name collisions</li></ul> | <ul><li>`this.xxx` all over the place</li><li>more complex implementation</li></ul> | <ul><li>`state.xxx` all over the place</li><li>inconsistent syntax</li></ul> |
