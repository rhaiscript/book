Run on Embedded Devices
=======================

{{#include ../../links.md}}


```admonish tip.side.wide "Tip: Ahead-of-Time"
Scripts intended for embedded environments are seldom dynamic &ndash; they are usually prepared
ahead of time This allows the use of AOT (ahead-of-time) techniques to reduce resource usage.
```

Embedded devices often have tight limits on both program size and runtime memory (RAM).

The following techniques can reduce Rhai's footprint on such a device, especially when scripts are
prepared ahead of time.


Run Rhai Grain Bytecodes
------------------------

The standard interpreter evaluates a script via its [`AST`], which is much larger than the
source text must all be kept in memory.

Instead, use [Rhai Grain] to transpile scripts into [bytecodes] and execute them with the [VM].

The [bytecode][bytecodes-wiki] instruction stream is a compact representation of the script's [`AST`],
avoiding the need to keep it on the device.

Scripts can be transpiled separately on a development or build machine, and the [bytecodes] shipped
to the device for execution.

Enable the [`grain`] feature for [Rhai Grain].


Build with `no_ast`
-------------------

When scripts are transpiled to [Rhai Grain] [bytecodes], enable [`no_ast`] in the device build.

This removes the [`AST`], tokenizer, parser and [optimizer][script optimization],
significantly reducing the runtime's program size.

The device can only run [bytecodes]; it cannot compile or evaluate Rhai source text itself.


Lazy-Load the Standard Library
------------------------------

```admonish info.side "See Also"
See [lazy-loading][lazy] for details.
```

Use [`Engine::new_raw`][raw `Engine`] with `StandardPackage::new_lazy` to avoid eagerly loading the
entire standard library:

Functions are looked up only when called, which can significantly reduce runtime heap usage on
memory-constrained devices (lower by 15-20x). The trade-off is slower resolution the first time a function is used.

```rust
let mut engine = Engine::new_raw();

StandardPackage::new_lazy().register_into_engine(&mut engine);
//               ^^^^^^^^^^ lazy-loaded Standard library
```

```admonish warning.small "Warming: Strings Interner"

`Engine::new_raw` also disables the [strings interner] by default. If scripts create many [strings],
consider enabling it with [`Engine::set_max_strings_interned`][options].
```
