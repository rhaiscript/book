Infinite Loop
=============

{{#include ../links.md}}

Infinite loops follow Rust syntax.

Like Rust, `continue` can be used to skip to the next iteration, by-passing all following statements;
`break` can be used to break out of the loop unconditionally.

~~~admonish tip.small "Tip: Disable `loop`"

`loop` can be disabled via [`Engine::set_allow_looping`][options].
~~~

```rust
let x = 10;

loop {
    x -= 1;

    if x > 5 { continue; }  // skip to the next iteration

    print(x);

    if x == 0 { break; }    // break out of loop
}
```

~~~admonish danger.small "Remember the `break` statement"

A `loop` statement without a `break` statement inside its loop block is infinite.
There is no way for the loop to stop iterating.
~~~


Loop Expression
---------------

Like Rust, `loop` statements can also be used as _expressions_.

The `break` statement takes an optional expression that provides the return value.

The default return value of a `loop` expression is [`()`].

~~~admonish tip.small "Tip: Disable all loop expressions"

Loop expressions can be disabled via [`Engine::set_allow_loop_expressions`][options].
~~~

```js
let x = 0;

// 'loop' can be used just like an expression
let result = loop {
    if is_magic_number(x) {
        // if the loop breaks here, return a specific value
        break get_magic_result(x);
    }

    x += 1;

    // ... if the loop exits here, the return value is ()
};

if result == () {
    print("Magic number not found!");
} else {
    print(`Magic result = ${result}!`);
}
```
