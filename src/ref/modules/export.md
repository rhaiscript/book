Export Variables, Functions and Sub-Modules From a Script
=========================================================

The easiest way to expose a collection of [functions](../functions.md) as a self-contained [module](index.md)
is to do it via a Rhai script itself.

The script text is evaluated.

[Variables](../variables.md) are then selectively exposed via the `export` statement.

[Functions](../functions.md) defined by the script are automatically exported, unless marked as `private`.

Modules loaded within this [module](index.md) at the global level become _sub-modules_ and are also
automatically exported.


Export Global Constants
-----------------------

The `export` statement, which can only be at global level, exposes a selected
[variable](../variables.md) as member of a [module](index.md).

[Variables](../variables.md) not exported are _private_ and hidden. They are merely used to
initialize the [module](index.md), but cannot be accessed from outside.

Everything exported from a [module](index.md) is **[constant](../constants.md)** (i.e. read-only).

```js
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

```admonish tip.small "Tip: Multiple exports"

[Variables] can be exported under multiple names.
For example, the following exports three [variables]:
* `x` as `x` and `hello`
* `y` as `foo` and `bar`
* `z` as `z`

~~~js
export x;
export x as hello;
export y as foo;
export x as world;
export y as bar;
export z;
~~~
```

```admonish bug.small "Do not export closures"

A [function pointer](../fn-ptr.md), [anonymous function](../fn-anon.md) or [closure](../fn-closure.md),
is not a _first-class function_.

It is _syntactic sugar_ only, capturing the _name_ of a [function](../functions.md) to call.

Exporting them causes a runtime error.
```


Export Functions
----------------

```admonish info.side.wide "Private functions"

`private` [functions](../functions.md) are commonly called within the [module](index.md) only.
They cannot be accessed otherwise.
```

All [functions](../functions.md) are automatically exported, _unless_ it is explicitly opt-out with
the `private` prefix.

[Functions](../functions.md) declared `private` are hidden to the outside.

```rust
// This is a module script.

fn inc(x) { x + 1 }     // script-defined function - default public

private fn foo() {}     // private function - hidden
```


Sub-Modules
-----------

All loaded [modules](index.md) are automatically exported as sub-modules.

~~~admonish tip.small "Tip: Skip exporting a module"

To prevent a [module](index.md) from being exported, load it inside a block statement
so that it goes away at the end of the block.

```js
// This is a module script.

import "hello" as foo;      // <- exported

{
    import "world" as bar;  // <- not exported
}
```
~~~
