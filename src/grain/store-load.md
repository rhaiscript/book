Store Bytecodes and Reload It Later
===================================

{{#include ../links.md}}


```admonish note.side "Benefits"
The size of the [bytecodes] stream is usually smaller than the original script source,
and _much_ smaller than memory required to hold the entire [`AST`].
```

This is the primary use case for Rhai Grain: transpile an [`AST`] to [bytecodes]
on a host, then run it on a resource-constrained separate target that does not have
access to the Rhai parser or the source file.

Once a script has been transpiled into [bytecodes] all the way through, it can be saved
into whatever storage (e.g. a file) as a dense byte stream.

The [bytecodes], when loaded, can be consumed by another `Vm` and can run with a fresh
[`Engine`] instance.


Example
-------

### Transpile to bytecodes

```rust
use rhai::grain::{Compiler, Program, Vm};

let engine = Engine::new();
let ast = engine.compile(script)?;

let grain = Compiler::new();
let bytecodes = grain.compile(&ast);

// Make sure that there are no unsupported features in the script.
assert_eq!(bytecodes.residual_count(), 0);
```

```admonish warning.small "Residual AST fragments"
The important check here is `bytecodes.residual_count()`.

If Rhai Grain cannot transpile the script completely (i.e. if the script depends on certain
unsupported functionality such as [`eval`]), the program will still contain _residual_
[`AST`] fragments and `write` will refuse to serialize it.

In practice, this is a good sanity check: if the [bytecodes] can be written out, it no longer
depends on the script source at all (nor the [`AST`]).
```

### Write out to disk

```rust
// Serialize the bytecodes into a dense byte stream.
let buf = bytecodes.write()?;

// Save it to a file.
std::fs::write("my_program.rgrn", buffer)?;
```

### Reload from disk and run

```rust
// Load the bytecodes from disk.
let buf = std::fs::read("my_program.rgrn")?;

// Deserialize the bytecodes.
let loaded = Program::read(&buf)?;

let mut vm = Vm::new(&engine);
let result = vm.eval(&loaded)?;
```

~~~admonish warning.small "Still needs an `Engine`"
The [bytecodes] is not a standalone executable.

It still must be evaluated by a `Vm`, which still uses an actual Rhai [`Engine`]
to call registered native functions and perform other runtime operations.

This is why Rhai Grain is particularly useful for deployment scenarios where
the target device should _not_ have access to the script source, but still must know
what functions and behavior are legal to call.

The [bytecodes] is a compact representation of the [`AST`]; the runtime is still Rhai.
~~~
