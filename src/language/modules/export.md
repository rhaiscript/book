Export Variables, Functions and Sub-Modules
==========================================

{{#include ../../links.md}}


The easiest way to expose a collection of functions as a self-contained [module] is to do it via a Rhai script itself.

See the section on [_Creating a Module from AST_]({{rootUrl}}/rust/modules/ast.md) for more details.

The script text is evaluated, variables are then selectively exposed via the [`export`] statement.
Functions defined by the script are automatically exported.

Modules loaded within this module at the global level become _sub-modules_ and are also automatically exported.


Export Global Variables
----------------------

The `export` statement, which can only be at global level, exposes selected variables as members of a module.

Variables not exported are _private_ and hidden. They are merely used to initialize the module,
but cannot be accessed from outside.

Everything exported from a module is **constant** (i.e. read-only).

```
// This is a module script.

let hidden = 123;       // variable not exported - default hidden
let x = 42;             // this will be exported below

export x;               // the variable 'x' is exported under its own name

export const x = 42;    // convenient short-hand to declare a constant and export it
                        // under its own name

export let x = 123;     // variables can be exported as well, though it'll still be constant

export x as answer;     // the variable 'x' is exported under the alias 'answer'
                        // another script can load this module and access 'x' as 'module::answer'

{
    let inner = 0;      // local variable - it disappears when the statement block ends,
                        //                  therefore it is not 'global' and cannot be exported

    export inner;       // <- syntax error: cannot export a local variable
}
```

### Multiple Exports

One `export` statement can export multiple variables, even under multiple names.

```
// The following exports three variables:
//   - 'x' as 'x' and 'hello'
//   - 'y' as 'foo' and 'bar'
//   - 'z' as 'z'
export x, x as hello, x as world, y as foo, y as bar, z;
```


Export Functions
----------------

All functions are automatically exported, _unless_ it is explicitly opt-out with the [`private`] prefix.

Functions declared [`private`] are hidden to the outside.

```rust,no_run
// This is a module script.

fn inc(x) { x + 1 }     // script-defined function - default public

private fn foo() {}     // private function - hidden
```

[`private`] functions are commonly called to initialize the module.
They cannot be called apart from this.


Sub-Modules
-----------

All loaded modules are automatically exported as sub-modules.

To prevent a module from being exported, load it inside a block statement so that it goes away at the
end of the block.

```
// This is a module script.

import "hello" as foo;      // exported as sub-module 'foo'

{
    import "world" as bar;  // not exported - the module disappears at the end
                            //                of the statement block and is not 'global'
}
```
