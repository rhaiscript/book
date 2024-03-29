`FileModuleResolver`
====================

{{#include ../../../links.md}}


```admonish abstract.small "Default"

`FileModuleResolver` is the default for [`Engine::new`][`Engine`].
```

The _default_ [module] resolution service, not available for [`no_std`] or [WASM] builds.
Loads a script file (based off the current directory or a specified one) with `.rhai` extension.


Function Namespace
-------------------

All functions in the [_global_ namespace][function namespace], plus all those defined in the same
[module], are _merged_ into a _unified_ [namespace][function namespace].

All [modules] imported at _global_ level via [`import`] statements become sub-[modules],
which are also available to [functions] defined within the same script file.


Base Directory
--------------

```admonish tip.side.wide "Tip: Default"

If the base directory is not set, then relative paths are based off the directory of the loading script.

This allows scripts to simply cross-load each other.
```

_Relative_ paths are resolved relative to a _root_ directory, which is usually the base directory.

The base directory can be set via `FileModuleResolver::new_with_path` or
`FileModuleResolver::set_base_path`.


Custom [`Scope`]
----------------

```admonish tip.side.wide "Tip"

This [`Scope`] can conveniently hold global [constants] etc.
```

The `set_scope` method adds an optional [`Scope`] which will be used to [optimize][script optimization] [module] scripts.


Caching
-------

```admonish tip.side.wide "Tip: Enable/disable caching"

Use `enable_cache` to enable/disable the cache.
```

By default, [modules] are also _cached_ so a script file is only evaluated _once_, even when
repeatedly imported.


Unix Shebangs
-------------

On Unix-like systems, the _shebang_ (`#!`) is used at the very beginning of a script file to mark a
script with an interpreter (for Rhai this would be [`rhai-run`]({{rootUrl}}/start/bin.md)).

If a script file starts with `#!`, the entire first line is skipped.
Because of this, Rhai scripts with shebangs at the beginning need no special processing.

```js
#!/home/to/me/bin/rhai-run

// This is a Rhai script

let answer = 42;
print(`The answer is: ${answer}`);
```


Example
-------


```rust
┌────────────────┐
│ my_module.rhai │
└────────────────┘

// This function overrides any in the main script.
private fn inner_message() { "hello! from module!" }

fn greet() {
    print(inner_message());     // call function in module script
}

fn greet_main() {
    print(main_message());      // call function not in module script
}


┌───────────┐
│ main.rhai │
└───────────┘

// This function is overridden by the module script.
fn inner_message() { "hi! from main!" }

// This function is found by the module script.
fn main_message() { "main here!" }

import "my_module" as m;

m::greet();                     // prints "hello! from module!"

m::greet_main();                // prints "main here!"
```


Simulate Virtual Functions
--------------------------

When calling a namespace-qualified [function] defined within a [module], other [functions] defined
within the same module override any similar-named [functions] (with the same number of parameters)
defined in the [global namespace][function namespace].

This is to ensure that a [module] acts as a self-contained unit and [functions] defined in the
calling script do not override [module] code.

In some situations, however, it is actually beneficial to do it in reverse: have [module] [functions] call
[functions] defined in the calling script (i.e. in the [global namespace][function namespace]) if
they exist, and only call those defined in the [module] if none are found.

One such situation is the need to provide a _default implementation_ to a simulated _virtual_ function:

```rust
┌────────────────┐
│ my_module.rhai │
└────────────────┘

// Do not do this (it will override the main script):
// fn message() { "hello! from module!" }

// This function acts as the default implementation.
private fn default_message() { "hello! from module!" }

// This function depends on a 'virtual' function 'message'
// which is not defined in the module script.
fn greet() {
    if is_def_fn("message", 0) {    // 'is_def_fn' detects if 'message' is defined.
        print(message());
    } else {
        print(default_message());
    }
}


┌───────────┐
│ main.rhai │
└───────────┘

// The main script defines 'message' which is needed by the module script.
fn message() { "hi! from main!" }

import "my_module" as m;

m::greet();                         // prints "hi! from main!"


┌────────────┐
│ main2.rhai │
└────────────┘

// The main script does not define 'message' which is needed by the module script.

import "my_module" as m;

m::greet();                         // prints "hello! from module!"
```

