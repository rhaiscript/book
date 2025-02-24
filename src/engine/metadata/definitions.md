Generate Definition Files for Language Server
=============================================

{{#include ../../links.md}}

Rhai's [language server][lsp] works with IDEs to provide integrated support for the Rhai scripting language.

Functions and [modules] registered with an [`Engine`] can output their [metadata][functions metadata]
into _definition files_ which are used by the [language server][lsp].

Definitions are generated via the `Engine::definitions` and `Engine::definitions_with_scope` API.

This API requires the [`metadata`] and [`internals`] feature.


Configurable Options
--------------------

The `Definitions` type supports the following options in a fluent method-chaining style.

| Option                                                              | Method                      | Default |
| ------------------------------------------------------------------- | --------------------------- | :-----: |
| Write headers in definition files?                                  | `with_headers`              | `false` |
| Include [standard packages][built-in packages] in definition files? | `include_standard_packages` | `true`  |

```rust
engine
    .definitions()
    .with_headers(true)                     // write headers in all files
    .include_standard_packages(false)       // skip standard packages
    .write_to_dir("path/to/my/definitions")
    .unwrap();
```


Example
-------

```rust
use rhai::{Engine, Scope};
use rhai::plugin::*;

// Plugin module: 'general_kenobi'
#[export_module]
pub mod general_kenobi {
    use std::convert::TryInto;

    /// Returns a string where "hello there" is repeated 'n' times.
    pub fn hello_there(n: i64) -> String {
        "hello there ".repeat(n.try_into().unwrap())
    }
}

// Create scripting engine
let mut engine = Engine::new();

// Create custom Scope
let mut scope = Scope::new();

// This variable will also show up in the generated definition file.
scope.push("hello_there", "hello there");

// Static module namespaces will generate independent definition files.
engine.register_static_module(
        "general_kenobi",
        exported_module!(general_kenobi).into()
);

// Custom operators will also show up in the generated definition file.
engine.register_custom_operator("minus", 100).unwrap();
engine.register_fn("minus", |a: i64, b: i64| a - b);

engine.run_with_scope(&mut scope,
        "hello_there = general_kenobi::hello_there(4 minus 2);"
)?;

// Output definition files in the specified directory.
engine
    .definitions()
    .write_to_dir("path/to/my/definitions")
    .unwrap();

// Output definition files in the specified directory.
// Variables in the provided 'Scope' are included.
engine
    .definitions_with_scope(&scope)
    .write_to_dir("path/to/my/definitions")
    .unwrap();

// Output a single definition file with everything merged.
// Variables in the provided 'Scope' are included.
engine
    .definitions_with_scope(&scope)
    .write_to_file("path/to/my/definitions/all_in_one.d.rhai")
    .unwrap();

// Output functions metadata to a JSON string.
// Functions in standard packages are skipped and not included.
let json = engine
    .definitions()
    .include_standard_packages(false)   // skip standard packages
    .unwrap();
```


Definition Files
----------------

The generated definition files will look like the following.

```rust
┌───────────────────────┐
│ general_kenobi.d.rhai │
└───────────────────────┘

module general_kenobi;

/// Returns a string where "hello there" is repeated 'n' times.
fn hello_there(n: int) -> String;


┌──────────────────┐
│ __scope__.d.rhai │
└──────────────────┘

module static;

let hello_there;


┌───────────────────┐
│ __static__.d.rhai │
└───────────────────┘

module static;

op minus(int, int) -> int;

        :
        :


┌────────────────────┐
│ __builtin__.d.rhai │
└────────────────────┘

module static;

        :
        :


┌──────────────────────────────┐
│ __builtin-operators__.d.rhai │
└──────────────────────────────┘

module static;

        :
        :

```


All-in-One Definition File
--------------------------

`Definitions::write_to_file` generates a single definition file with everything merged in, like the following.

```rust
module static;

op minus(int, int) -> int;

        :
        :

module general_kenobi {
    /// Returns a string where "hello there" is repeated 'n' times.
    fn hello_there(n: int) -> String;
}

let hello_there;
```
