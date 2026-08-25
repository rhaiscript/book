Diagnostics
===========

{{#include ../links.md}}


When errors occur, Rhai Grain provides the exact same level of diagnostics as [`Engine`] does
&ndash; i.e., full source positions, stack traces, and error messages.

Diagnostics information, however, is a heavy payload, sometimes even larger than the instructions
and data.  Rhai Grain allows stripping the diagnostics information from [bytecodes] and keeping it
in a separate [`Sidecar`] that resides on the host together with the script source.


Stripping Bytecodes
-------------------

For an even more compact shipping format, Rhai Grain also supports a stripped form of
the serialized [bytecodes] that removes diagnostics information.

```rust
use rhai::grain::{Compiler, Program};

let engine = Engine::new();
let ast = engine.compile(script)?;

let grain = Compiler::new();
let bytecodes = grain.compile(&ast);

// Make sure that there are no unsupported features in the script.
assert_eq!(bytecodes.residual_count(), 0);

// Serialize the bytecodes into a dense byte stream with no diagnostics.
let stripped = bytecodes.write_stripped()?;
```


Keep Diagnostics on the Host
----------------------------

The Rhai Grain `Sidecar` keeps source positions and debug information separately
so the target device can carry only the [bytecodes] payload.

```rust
use rhai::grain::{Compiler, Program, Vm};

let bytecodes = Program::read(&stripped.artifact).unwrap();

let mut vm = Vm::new(&engine);
let error = vm.eval_with_scope(&mut Scope::new(), &bytecodes).unwrap_err();

let sites = stripped.sidecar.resolve(&vm.fault_trace());

assert!(error.position().is_none());
assert_eq!(sites[0].unwrap().line, 1);
```
