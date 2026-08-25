Primary API
===========

{{#include ../links.md}}

The relevant types live under `rhai::grain` namespace.

```rust
use rhai::grain::{Compiler, Program, Vm};
```

| Method                                                    | Description                                                                                         |
| --------------------------------------------------------- | --------------------------------------------------------------------------------------------------- |
| `Compiler::new` \| `Compiler::compile`                    | transpile an [`AST`] into Rhai Grain [bytecodes].                                                   |
| `Program::write` \| `Program::read`                       | serialize and reload a chunk of Rhai Grain [bytecodes].                                             |
| `Program::write_stripped`                                 | remove diagnostics from the Rhai Grain [bytecodes].                                                 |
| `Program::residual_count` \| `Program::first_unsupported` | report features not yet supported.                                                                  |
| `Vm::eval` \| `Vm::eval_with_scope`                       | run a script transpiled into Rhai Grain [bytecodes].                                                |
| `Vm::eval_with_callback`                                  | run a script transpiled into Rhai Grain [bytecodes] with _wrappers_ for script-defined [functions]. |
| `Vm::fault_trace`                                         | retrieve the trace after an error.                                                                  |
| `Sidecar::resolve`                                        | map fault traces back to source locations.                                                          |
