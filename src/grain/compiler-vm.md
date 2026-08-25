Rhai Grain Transpiler & VM
==========================

{{#include ../links.md}}

Rhai's default execution path is an [`AST`]-walking interpreter.
It is flexible and easy to use, but also keeps a full [`AST`] tree in memory.

Interpretation is also slower than a script transpiled into [bytecodes][bytecodes-wiki] and
executed in a [virtual machine](https://en.wikipedia.org/wiki/Virtual_Machine) (VM).

Rhai Grain replaces the [`AST`] with a flat stream of instructions, resulting in a compact
representation that can be produced (transpiled) on one machine and loaded onto another,
completely different, machine without carrying the original script source text.

```admonish note.small "Runtime Behavior"
Rhai Grain still uses Rhai's normal runtime services, including registered functions and [modules].

The VM attempts to reproduce the Rhai [`Engine`]'s behavior as faithfully as possible
so script runs are predictable.
```


Examples
--------

Compile a script into [`AST`], then further transpile it into [bytecodes][bytecodes-wiki] using
`rhai::grain::Compiler`.

The [bytecodes][bytecodes-wiki] can be executed using `rhai::grain::Vm`.

### Transpile a script into bytecodes and execute it

```rust
use rhai::grain::{Compiler, Vm};

// Create new Engine.
let engine = Engine::new();

// Compile script to AST.
let ast = engine.compile(script)?;

// Create Rhai Grain transpiler.
let grain = Compiler::new();

// Transpile AST to bytecodes.
let bytecodes = grain.compile(&ast);

// The bytecodes can be written to a file or sent over the network etc.,
// then loaded later, even by a completely different Rhai program
// in a completely different computer!

// Execute bytecodes using Rhai Grain VM.
let mut vm = Vm::new(&engine);

// The result is exactly the same as evaluated via`engine.eval_ast`.
let result = vm.eval(&bytecodes)?;
```

The [`Scope`] is the same for Rhai. A script declares variables into the [`Scope`]
in exactly the manner as it would when evaluated through `Engine::eval_with_scope`.
