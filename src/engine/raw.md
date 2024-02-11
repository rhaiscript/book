Raw `Engine`
============

{{#include ../links.md}}

`Engine::new` creates a scripting [`Engine`] with common functionalities (e.g. printing to `stdout`
via [`print`] or [`debug`]).

In many controlled embedded environments, however, these may not be needed and unnecessarily occupy
application code storage space.

```admonish info.side.wide "Built-in operators"

Even with a raw [`Engine`], some operators are built-in and always available.

See [_Built-in Operators_][built-in operators] for a full list.
```

Use `Engine::new_raw` to create a _raw_ [`Engine`], in which only a minimal set of
[built-in][built-in operators] basic arithmetic and logical operators are supported.

To add more functionalities to a _raw_ [`Engine`], load [packages] into it.

Since [packages] can be _shared_, this is an extremely efficient way to create multiple instances of
the same [`Engine`] with the same set of functions.

|                       |    `Engine::new`     | `Engine::new_raw` |
| --------------------- | :------------------: | :---------------: |
| [Built-in operators]  |         yes          |        yes        |
| [Package] loaded      |  `StandardPackage`   |      _none_       |
| [Module resolver]     | `FileModuleResolver` |      _none_       |
| [Strings interner]    |         yes          |       _no_        |
| [`on_print`][`print`] |         yes          |      _none_       |
| [`on_debug`][`debug`] |         yes          |      _none_       |


```admonish warning.small "Warning: No strings interner"

A _raw_ [`Engine`] disables the _[strings interner]_ by default.

This may lead to a significant increase in memory usage if many strings are created in scripts.

Turn the _[strings interner]_ back on via [`Engine::set_max_strings_interned`][options].
```

~~~admonish example "`Engine::new` is equivalent to..."
```rust
use rhai::module_resolvers::FileModuleResolver;
use rhai::packages::StandardPackage;

// Create a raw scripting Engine
let mut engine = Engine::new_raw();

// Use the file-based module resolver
engine.set_module_resolver(FileModuleResolver::new());

// Enable the strings interner
engine.set_max_strings_interned(1024);

// Default print/debug implementations
engine.on_print(|text| println!("{text}"));

engine.on_debug(|text, source, pos| match (source, pos) {
    (Some(source), crate::Position::NONE) => println!("{source} | {text}"),
    (Some(source), pos) => println!("{source} @ {pos:?} | {text}"),
    (None, crate::Position::NONE) => println!("{text}"),
    (None, pos) => println!("{pos:?} | {text}"),
});

// Register the Standard Package
let package = StandardPackage::new();

// Load the package into the [`Engine`]
package.register_into_engine(&mut engine);
```
~~~
