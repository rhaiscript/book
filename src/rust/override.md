Override a Built-in Function
===========================

{{#include ../links.md}}

Any similarly-named function defined in a script _overrides_ any built-in or registered
native Rust function of the same name and number of parameters.

```rust
// Override the built-in function 'to_float' when called as a method
fn to_float() {
    print("Ha! Gotcha! " + this);
    42.0
}

let x = 123.to_float();

print(x);       // what happens?
```

A registered native Rust function, in turn, overrides any built-in function of the
same name, number and types of parameters.
