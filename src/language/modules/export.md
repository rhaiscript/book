Export Variables, Functions and Sub-Modules From a Script
========================================================

{{#include ../../links.md}}

```admonish info.side "See also"

See [_Create a Module from AST_]({{rootUrl}}/rust/modules/ast.md) for more details.
```

The easiest way to expose a collection of [functions] as a self-contained [module] is to do it via a Rhai script itself.

The script text is evaluated.

[Variables] are then selectively exposed via the [`export`] statement.

[Functions] defined by the script are automatically exported, unless marked as [private][`private`].

Modules loaded within this [module] at the global level become _sub-modules_ and are also automatically exported.


Export Global Variables
----------------------

The `export` statement, which can only be at global level, exposes a selected [variable] as member of a [module].

[Variables] not exported are _private_ and hidden. They are merely used to initialize the [module],
but cannot be accessed from outside.

Everything exported from a [module] is **[constant]** (i.e. read-only).

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

```admonish tip "Tip: Multiple exports"

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

~~~admonish danger "Do not export closures"

A [function pointer], [anonymous function] or [closure], is not a _first-class function_.
It is _syntactic sugar_ only, capturing the _name_ of a [function] to call.

The actual [function] may not reside in the appropriate [namespace][function namespace],
so the call will fail.

```js
┌────────────────┐
│ my_module.rhai │
└────────────────┘

fn increment(x) {
    x + 1
}

export let inc = Fn("increment");   // exports a function pointer


┌───────────┐
│ main.rhai │
└───────────┘

import "my_module" as my_mod;

print(my_mod::increment(41));       // ok!

let x = my_mod::inc.call(41);       // runtime error:
                                    //    function 'increment' not found
```
~~~


Export Functions
----------------

```admonish info.side.wide "Private functions"

[`private`] [functions] are commonly called to initialize the [module].
They cannot be accessed otherwise.
```

All [functions] are automatically exported, _unless_ it is explicitly opt-out with the [`private`] prefix.

[Functions] declared [`private`] are hidden to the outside.

```rust,no_run
// This is a module script.

fn inc(x) { x + 1 }     // script-defined function - default public

private fn foo() {}     // private function - hidden
```


Sub-Modules
-----------

All loaded [modules] are automatically exported as sub-modules.

To prevent a [module] from being exported, load it inside a block statement so that it goes away at the
end of the block.

```js
// This is a module script.

import "hello" as foo;      // exported as sub-module 'foo'

{
    import "world" as bar;  // not exported - the module disappears at the end
                            //                of the statement block and is not 'global'
}
```
