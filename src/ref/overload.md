Function Overloading
====================

{{#title Function Overloading}}

[Functions](functions.md) defined in script can be _overloaded_ by _arity_ (i.e. they are resolved
purely upon the function's _name_ and _number_ of parameters, but not parameter _types_ since all
parameters are dynamic).

New definitions _overwrite_ previous definitions of the same name and number of parameters.

```js
fn foo(x, y, z) {
    print(`Three!!! ${x}, ${y}, ${z}`);
}
fn foo(x) {
    print(`One! ${x}`);
}
fn foo(x, y) {
    print(`Two! ${x}, ${y}`);
}
fn foo() {
    print("None.");
}
fn foo(x) {     // <- overwrites previous definition
    print(`HA! NEW ONE! ${x}`);
}

foo(1,2,3);     // prints "Three!!! 1,2,3"

foo(42);        // prints "HA! NEW ONE! 42"

foo(1,2);       // prints "Two!! 1,2"

foo();          // prints "None."
```
