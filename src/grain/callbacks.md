Run Bytecodes with Callback
===========================

{{#include ../links.md}}


The standard `Vm::eval` and `Vm::eval_with_scope` path is enough for ordinary scripts.

A Rhai Grain [bytecodes] program becomes a special case when it creates or receives a
[function pointer] that a native Rust function later calls.


`Vm::eval_with_callbacks`
-------------------------

This is the purpose of `Vm::eval_with_callbacks`: it registers a native _wrapper_ for each
transpiled function for the duration of the run, so a script that creates a [closure] and then
hands it to a host callback can still be resolved by Rhai.

This is necessary when the script does something like:

```rhai
let values = [1, 2, 3];     // an array

values.map(|x| x * 2);      // native function 'map' taking closure as callback
```

The [`Engine`] must be able to resolve that [function pointer] within the native Rust call.

```admonish warning.small "Callbacks are necessary for closures"
Without callback wrappers, Rhai Grain would know the clos7ure [function] exists in the [bytecodes],
but the [`Engine`] that runs the native function (`map`) would not.
```


Run with Callback Wrappers
--------------------------

```rust
use rhai::grain::{Compiler, Vm};

let engine = Engine::new();
let ast = engine.compile(script)?;

// Transpile into shared bytecodes.
let grain = Compiler::new();
let bytecodes = grain.compile(&ast).into_shared();

// Use 'eval_with_callbacks'.
let mut vm = Vm::new(&engine);
let value = vm.eval_with_callbacks(&mut scope, &bytecodes)?;
```

```admonish tip.small "Tip: Use callbacks only when necessary"
The callback-enabled path is not needed for _every_ script.

If the script has no [function pointer] or [closure] usage,
plain `Vm::eval` is simpler and faster.

It becomes necessary only when Rhai must cross from native Rust
back into Rhai Grain through a [function pointer] or a [closure].
```
