Create a Module from an AST
==========================

{{#include ../../links.md}}


`Module::eval_ast_as_new`
------------------------

```admonish info.side.wide "Encapsulated environment"

`Module::eval_ast_as_new` encapsulates the entire [`AST`] into each function call, merging the
[module namespace][function namespace] with the [global namespace][function namespace].

Therefore, [functions] defined within the same [module] script can cross-call each other.
```

```admonish info.side.wide "See also"

See [_Export Variables, Functions and Sub-Modules from Script_][`export`] for details on how to prepare
a Rhai script for this purpose as well as to control which [functions]/[variables] to export.
```

A [module] can be created from a single script (or pre-compiled [`AST`]) containing global
[variables], [functions] and [sub-modules][module] via `Module::eval_ast_as_new`.

When given an [`AST`], it is first evaluated (usually to [import][`import`] [modules] and set up global
[constants] used by [functions]), then the following items are exposed as members of the new [module]:

* global [variables] and [constants] &ndash; all [variables] and [constants] exported via the
  [`export`] statement (those not exported remain hidden),

* [functions] not specifically marked [`private`],

* imported [modules] that remain in the [`Scope`] at the end of a script run become sub-modules.


Examples
--------

Don't forget the [`export`] statement, otherwise there will be no [variables] exposed by the
[module] other than non-[`private`] [functions] (unless that's intentional).

```rust
use rhai::{Engine, Module};

let engine = Engine::new();

// Compile a script into an 'AST'
let ast = engine.compile(
r#"
    // Functions become module functions
    fn calc(x) {
        x + add_len(x, 1)       // functions within the same module
                                // can always cross-call each other!
    }
    fn add_len(x, y) {
        x + y.len
    }

    // Imported modules become sub-modules
    import "another module" as extra;

    // Variables defined at global level can become module variables
    const x = 123;
    let foo = 41;
    let hello;

    // Variable values become constant module variable values
    foo = calc(foo);
    hello = `hello, ${foo} worlds!`;

    // Finally, export the variables and modules
    export x as abc;            // aliased variable name
    export foo;
    export hello;
"#)?;

// Convert the 'AST' into a module, using the 'Engine' to evaluate it first
// A copy of the entire 'AST' is encapsulated into each function,
// allowing functions in the module script to cross-call each other.
let module = Module::eval_ast_as_new(Scope::new(), &ast, &engine)?;

// 'module' now contains:
//   - sub-module: 'extra'
//   - functions: 'calc', 'add_len'
//   - constants: 'abc' (renamed from 'x'), 'foo', 'hello'
```
