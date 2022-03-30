Compile to a Self-Contained `AST`
================================

{{#include ../../links.md}}

```admonish tip.side "Tip"

It does not matter where the [`import`] statement occurs &mdash; e.g. deep within statement blocks
or within function bodies.
```

When a script [imports][`import`] external [modules] that may not be available later on, it is
possible to eagerly [_pre-resolve_][module resolver] these imports and embed them directly into a
self-contained [`AST`].

For instance, a system may periodically connect to a central source (e.g. a database) to load
scripts and compile them to [`AST`] form. Afterwards, in order to conserve bandwidth (or due to
other physical limitations), it is disconnected from the central source for self-contained
operation.

Compile a script into a _self-contained_ [`AST`] via `Engine::compile_into_self_contained`.

```rust
let mut engine = Engine::new();

// Compile script into self-contained AST using the current
// module resolver (default to `FileModuleResolver`) to pre-resolve
// 'import' statements.
let ast = engine.compile_into_self_contained(&mut scope, script)?;

// Make sure we can no longer resolve any module!
engine.set_module_resolver(DummyModuleResolver::new());

// The AST still evaluates fine, even with 'import' statements!
engine.run(&ast)?;
```

When such an [`AST`] is evaluated, [`import`] statements within are provided the _pre-resolved_
[modules] without going through the normal [module resolution][module resolver] process.


Only Static Paths
-----------------

`Engine::compile_into_self_contained` only pre-resolves [`import`] statements in the script
that are _static_, i.e. with a path that is a [string] literal.

```rust
// The following import is pre-resolved.
import "hello" as h;

if some_event() {
    // The following import is pre-resolved.
    import "hello" as h;
}

fn foo() {
    // The following import is pre-resolved.
    import "hello" as h;
}

// The following import is also pre-resolved because the expression
// is usually optimized into a single string during compilation.
import "he" + "llo" as h;

let module_name = "hello";

// The following import is NOT pre-resolved.
import module_name as h;
```
