Override a Built-in Function
===========================

{{#include ../links.md}}

Any similarly-named function defined in a script _overrides_ any built-in or registered
native Rust function of the same name and number of parameters.

```js
// Override the built-in function 'to_float' when called as a method
fn to_float() {
    print(`Ha! Gotcha! ${this}`);
    42.0
}

let x = 123.to_float();

print(x);       // what happens?
```


Search Order of Functions
-------------------------

Rhai searches for the correct implementation of a function in the following order:

* Rhai script-defined [functions],

* native Rust functions registered directly into the [`Engine`] via the `register_XXX` API,

* native Rust functions in [packages] that have been loaded into the [`Engine`] via `register_global_module`,

* native Rust or Rhai script-defined functions in [imported][`import`] [modules] that are exposed to
  the global [namespace][function namespace] (e.g. via the `#[rhai_fn(global)]` attribute in a
  [plugin module]),

* native Rust or Rhai script-defined functions in [modules] loaded via into the [`Engine`] via
  `register_static_module` that are exposed to the global [namespace][function namespace] (e.g. via
  the `#[rhai_fn(global)]` attribute in a [plugin module]),

* [built-in][built-in operators] functions.
