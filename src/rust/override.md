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

```admonish tip "Tip: Monkey patching Rhai"

Most of Rhai's built-in functionality resides in registered functions.

If you dislike any built-in function, simply provide your own implementation to
_override_ the built-in version.

The ability to modify the operating environment dynamically at runtime is called
"[monkey patching](https://en.wikipedia.org/wiki/Monkey_patch)."
It is rarely recommended, but if you need it, you need it bad.

In other words, do it only when _all else fails_.  Do not monkey patch Rhai simply
because you don't like the default functionality.
```


Search Order of Functions
-------------------------

Rhai searches for the correct implementation of a function in the following order:

* Rhai script-defined [functions],

* native Rust functions registered directly via the `Engine::register_XXX` API,

* native Rust functions in [packages] that have been loaded via `Engine::register_global_module`,

* native Rust or Rhai script-defined functions in [imported][`import`] [modules] that are exposed to
  the global [namespace][function namespace] (e.g. via the `#[rhai_fn(global)]` attribute in a
  [plugin module]),

* native Rust or Rhai script-defined functions in [modules] loaded via
  `Engine::register_static_module` that are exposed to the global [namespace][function namespace]
  (e.g. via the `#[rhai_fn(global)]` attribute in a [plugin module]),

* [built-in][built-in operators] functions.
