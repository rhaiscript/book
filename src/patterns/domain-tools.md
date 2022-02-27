Domain-Specific Tools
=====================

{{#include ../links.md}}

[bin tool]: {{rootUrl}}/start/bin.md
[bin tools]: {{rootUrl}}/start/bin.md

```admonish info "Usage scenario"

* A system has a _domain-specific_ API, requiring [custom types] and/or
  Rust [functions]({{rootUrl}}/rust/functions.md) to be registered and exposed to scripting.

* The system's behavior is controlled by Rhai script [functions], such as in the
  [_Scriptable Event Handler with State_](events.md), [_Control Layer_](control.md) or
  [_Singleton Command Object_](singleton.md) patterns.

* It is desirable to be able to _interactively_ test the system with different scripts,
  and/or [debug][debugger] them.
```

```admonish abstract "Key concepts"

* Leverage the pre-packaged [bin tools] &ndash; it is easy because each of them is
  one single source file.

* Modify the [`Engine`] creation code to include domain-specific registrations.
```


Implementation
--------------

### Copy necessary tool source files

Each [bin tool] is a single source file.

Download the necessary one(s) into the project's `bin` or `example` subdirectory.

| Select this source file                             | To make                               |
| --------------------------------------------------- | ------------------------------------- |
| [`rhai-run.rs`]({{repoHome}}/src/bin/rhai-run.rs)   | a simple script runner for the system |
| [`rhai-repl.rs`]({{repoHome}}/src/bin/rhai-repl.rs) | an interactive REPL for the system    |
| [`rhai-dbg.rs`]({{repoHome}}/src/bin/rhai-dbg.rs)   | a script debugger for the system      |

#### Example

```sh
rhai-run.rs -> /path/to/my_project/examples/test.rs
rhai-repl.rs -> /path/to/my_project/examples/repl.rs
rhai-dbg.rs -> /path/to/my_project/examples/db.rs
```

### Leverage `Engine` configuration code in the project

Assume the project already contains configuration code for a customized [`Engine`].

```rust,no_run
use rhai::Engine;
use rhai::plugin::*;

// Domain-specific data types
use my_project::*;

#[export_module]
mod my_domain_api {
            :
            :
    // plugin module
            :
            :
}

// This function creates a new Rhai scripting engine and
// configures it properly
fn create_scripting_engine(config: MySystemConfig) -> Engine {
    let mut engine = Engine::new();

    // Register domain-specific API into the Engine
    engine.register_type_with_name::<MyObject>("MyObject")
          .register_type_with_name::<MyOtherObject>("MyOtherObject")
          .register_fn(...)
          .register_fn(...)
          .register_fn(...)
                    :
                    :
        // register API functions
                    :
                    :
          .register_get_set(...)
          .register_index_get_set(...)
          .register_fn(...);

    // Plugin modules can be used to easily and quickly
    // register an entire API
    engine.register_global_module(exported_module!(my_domain_api));

    // Configuration options in 'MySystemConfig' may be used
    // to control the Engine's behavior
    if config.strict_mode {
        engine.set_strict_variables(true);
    }

    // Return the scripting engine
    engine
}
```

### Modify `Engine` creation

Each [bin tool] contains a line that creates the main script [`Engine`].

Modify it to call the project's creation function.

```rust,no_run
// Initialize scripting engine
let mut engine = Engine::new();

// Modify to this:
let mut engine = create_scripting_engine(my_config);
```

### Make sure that Rhai has the appropriate feature(s)

In the project's `Cargo.toml`, specify the Rhai dependency with `bin-features`.

```toml
[dependencies]
rhai = { version = "{{version}}", features = ["bin-features"] }
```

### Rebuild

Rebuild the project, which should automatically build all the customized tools.

### Run tools

Each customized tool now has access to the entire domain-specific API.
Functions and [custom types] can be used in REPL, [debugging][debugger] etc.
