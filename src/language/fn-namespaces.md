Function Namespaces
==================

{{#include ../links.md}}

Each Function is a Separate Compilation Unit
-------------------------------------------

[Functions] in Rhai are _pure_ and they form individual _compilation units_.
This means that individual functions can be separated, exported, re-grouped, imported,
and generally mix-'n-matched with other completely unrelated scripts.

For example, the `AST::merge` and `AST::combine` methods (or the equivalent `+` and `+=` operators)
allow combining all functions in one [`AST`] into another, forming a new, unified, group of functions.


Namespace Types
---------------

In general, there are two main types of _namespaces_ where functions are looked up:

| Namespace | How Many | Source                                                                                                                                                                                                                                        | Lookup                   | Sub-modules? | Variables? |
| --------- | :------: | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------ | :----------: | :--------: |
| Global    |   One    | 1) [`AST`] being evaluated<br/>2) `Engine::register_XXX` API<br/>3) global [modules] registered via `Engine::register_global_module`<br/>4) functions in static [modules] registered via `Engine::register_static_module` and marked _global_ | simple name              |   ignored    |  ignored   |
| Module    |   Many   | 1) [Module] registered via `Engine::register_static_module`<br/>2) [Module] loaded via [`import`] statement                                                                                                                                   | namespace-qualified name |     yes      |    yes     |


### Module Namespaces

There can be multiple module namespaces at any time during a script evaluation, usually loaded via the
[`import`] statement.

_Static_ module namespaces can also be registered into an [`Engine`] via `Engine::register_static_module`.

Functions and variables in module namespaces are isolated and encapsulated within their own environments.

They must be called or accessed in a _namespace-qualified_ manner.

```rust,no_run
import "my_module" as m;        // new module namespace 'm' created via 'import'

let x = m::calc_result();       // namespace-qualified function call

let y = m::MY_NUMBER;           // namespace-qualified variable/constant access

let z = calc_result();          // <- error: function 'calc_result' not found
                                //    in global namespace!
```


### Global Namespace

There is one _global_ namespace for every [`Engine`], which includes (in the following search order):

* All functions defined in the [`AST`] currently being evaluated.

* All native Rust functions and iterators registered via the `Engine::register_XXX` API.

* All functions and iterators defined in global [modules] that are registered into the [`Engine`] via
  `Engine::register_global_module`.

* Functions defined in [modules] registered via `Engine::register_static_module` that are specifically
  marked for exposure to the global namespace (e.g. via the `#[rhai(global)]` attribute in a [plugin module]).

Anywhere in a Rhai script, when a function call is made, the function is searched within the
global namespace, in the above search order.

Therefore, function calls in Rhai are _late_ bound &ndash; meaning that the function called cannot be
determined or guaranteed and there is no way to _lock down_ the function being called.
This aspect is very similar to JavaScript before ES6 modules.

```rust,no_run
// Compile a script into AST
let ast1 = engine.compile(r#"
    fn get_message() {
        "Hello!"                // greeting message
    }

    fn say_hello() {
        print(get_message());   // prints message
    }

    say_hello();
"#)?;

// Compile another script with an overriding function
let ast2 = engine.compile(r#"fn get_message() { "Boo!" }"#)?;

// Combine the two AST's
ast1 += ast2;                   // 'message' will be overwritten

engine.consume_ast(&ast1)?;     // prints 'Boo!'
```

Therefore, care must be taken when _cross-calling_ functions to make sure that the correct
functions are called.

The only practical way to ensure that a function is a correct one is to use [modules] -
i.e. define the function in a separate module and then [`import`] it:

```rust,no_run
+--------------+
| message.rhai |
+--------------+

fn get_message() { "Hello!" }


+-------------+
| script.rhai |
+-------------+

import "message" as msg;

fn say_hello() {
    print(msg::get_message());
}
say_hello();
```
