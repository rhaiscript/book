Automatic Global Module
=======================

{{#include ../links.md}}


When a [constant] is declared at global scope, it is added to a special [module] called `global`.

[Functions] can access those constants via the special `global` [module].

Naturally, the automatic `global` [module] is not available under [`no_function`].

```rust no_run
const CONSTANT = 42;        // this constant is automatically added to 'global'

{
    const INNER = 0;        // this constant is not at global level
}                           // <- it goes away here

fn foo(x) {
    x *= global::CONSTANT;  // ok! 'CONSTANT' exists in 'global'

    x * global::INNER       // <- error: constant 'INNER' not found in 'global'
}
```


Overriding `global`
-------------------

It is possible to _override_ the automatic global [module] by [importing][`import`] another [module]
under the name `global`.

```rust no_run
import "foo" as global;     // import a module as 'global'

const CONSTANT = 42;        // this constant is NOT added to 'global'

fn foo(x) {
    global::CONSTANT        // <- error: constant 'CONSTANT' not found in 'global'
}
```
