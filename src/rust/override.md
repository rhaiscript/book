Override a Built-in Function
===========================

{{#include ../links.md}}

Any similarly-named function defined in a script _overrides_ any built-in or registered
native Rust function of the same name and number of parameters.

```rust
// Override the built-in function 'to_float' when called as a method
fn to_float() {
    print("Ha! Gotcha! " + this);
    42.0
}

let x = 123.to_float();

print(x);       // what happens?
```


Search Order of Functions
-------------------------

Rhai searches for the correct implementation of a function in the following order:

* Rhai script-defined [functions].

* Native Rust functions registered directly into the [`Engine`] via the `Engine::register_XXX` API.

* Native Rust functions in [packages] that have been loaded.

* Native Rust or Rhai script-defined functions in [imported][`import`] [modules] (or [modules]
  loaded via `Engine::register_static_module`) that have are exposed globally (e.g. via the
  `#[rhai_fn(global)]` attribute in a [plugin module]).

* [Built-in][built-in operators] functions.
