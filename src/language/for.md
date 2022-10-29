For Loop
========

{{#include ../links.md}}

Iterating through a numeric [range] or an [array], or any type with a registered [type iterator],
is provided by the `for` ... `in` loop.

There are two alternative syntaxes, one including a counter variable:

> `for` _variable_ `in` _expression_ `{` ... `}`
>
> `for (` _variable_ `,` _counter_ `)` `in` _expression_ `{` ... `}`

~~~admonish tip.small "Tip: Disable `for` loops"

`for` loops can be disabled via [`Engine::set_allow_looping`][options].
~~~

Break or Continue
-----------------

Like C, `continue` can be used to skip to the next iteration, by-passing all following statements.

`break` can be used to break out of the loop unconditionally.


For Expression
--------------

Unlike Rust, `for` statements can also be used as _expressions_.

The `break` statement takes an optional expression that provides the return value.

The default return value of a `for` expression is [`()`].

~~~admonish tip.small "Tip: Disable all loop expressions"

Loop expressions can be disabled via [`Engine::set_allow_loop_expressions`][options].
~~~

```js
let a = [42, 123, 999, 0, true, "hello", "world!", 987.6543];

// 'for' can be used just like an expression
let index = for (item, count) in a {
    // if the 'for' loop breaks here, return a specific value
    switch item.type_of() {
        "i64" if item.is_even => break count,
        "f64" if item.to_int().is_even => break count,
    }

    // ... if the 'for' loop exits here, the return value is ()
};

if index == () {
    print("Magic number not found!");
} else {
    print(`Magic number found at index ${index}!`);
}
```


Counter Variable
----------------

The counter variable, if specified, starts from zero, incrementing upwards.

```js , no_run
let a = [42, 123, 999, 0, true, "hello", "world!", 987.6543];

// Loop through the array
for (item, count) in a {
    if x.type_of() == "string" {
        continue;                   // skip to the next iteration
    }

    // 'item' contains a copy of each element during each iteration
    // 'count' increments (starting from zero) for each iteration
    print(`Item #${count + 1} = ${item}`);

    if x == 42 { break; }           // break out of for loop
}
```
