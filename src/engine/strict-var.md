Strict Variables Mode
=====================

{{#include ../links.md}}

~~~admonish tip.side "`Scope` constants"

[Constants] in the external [`Scope`], when provided, count as definition.
~~~

By default, Rhai looks up access to [variables] from the enclosing block scope,
working its way outwards until it reaches the top (global) level, then it
searches the [`Scope`] (if any) that is passed into the `Engine::eval_with_scope` call.

Setting [`Engine::set_strict_variables`][options] to `true` turns on _Strict Variables Mode_,
which requires that:

* all [variables]/[constants] be defined within the same script before use,
  or they must be [variables]/[constants] within the provided [`Scope`] (if any),
* [modules] must be [imported][`import`], also within the same script, before use.

Within _Strict Variables_ mode, any attempt to access a [variable] or [module] before
definition/[import][`import`] results in a parse error.

This way, variable access errors (usually typos) are caught during compile time instead of runtime.

```rust
let x = 42;

let y = x * z;          // <- parse error under strict variables mode:
                        //    variable 'z' is not yet defined

let z = x + w;          // <- parse error under strict variables mode:
                        //    variable 'w' is undefined

foo::bar::baz();        // <- parse error under strict variables mode:
                        //    module 'foo' is not yet defined

fn test1() {
    foo::bar::baz();    // <- parse error under strict variables mode:
                        //    module 'foo' is defined
}

import "my_module" as foo;

foo::bar::baz();        // ok!

print(foo::xyz);        // ok!

let x = abc::def;       // <- parse error under strict variables mode:
                        //    module 'abc' is undefined

fn test2() {
    foo:bar::baz();     // ok!
}
```


```admonish question "TL;DR &ndash; Why isn't there a _Strict Functions_ mode?"

Why can't function calls be checked for validity as well?

Rust functions in Rhai can be [overloaded][function overloading]. This means that multiple versions of
the same Rust function can exist under the same name, each accepting different numbers and/or types
of arguments.

While it is possible to check, at compile time, whether a [variable] has been previously declared,
it is impossible to predict, at compile time, the _types_ of arguments to function calls, unless the
function in question takes no parameters.

Therefore, it is impossible to check, at compile time, whether a function call is valid given that
the types of arguments are unknown until runtime. QED.

Not to mention that it is also impossible to check for a function called via a [function pointer].
```
