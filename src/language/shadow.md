Variable Shadowing
==================

{{#include ../links.md}}

In Rhai, new [variables] automatically _shadow_ existing ones of the same name.  There is no error.

This behavior is consistent with Rust.

```rust,no_run
let x = 42;
let y = 123;

print(x);           // prints 42

let x = 88;         // <- 'x' is shadowed here

print(x);           // prints 88

let x = 0;          // <- 'x' is shadowed again

print(x);           // prints 0

{
    let x = 999;    // <- 'x' is shadowed in a block
    
    print(x);       // prints 999
}

print(x);           // prints 0 - shadowing within the block goes away

print(y);           // prints 123 - 'y' is not shadowed
```


Disable Shadowing
-----------------

If shadowing is not desired, set [`Engine::set_allow_shadowing`][options] to `false` to turn
[variables] shadowing off.

```rust,no_run
let x = 42;

let x = 123;        // <- syntax error: variable 'x' already defined
                    //    when variables shadowing is disallowed
```
