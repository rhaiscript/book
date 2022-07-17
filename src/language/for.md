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

Break or Continue
-----------------

Like C, `continue` can be used to skip to the next iteration, by-passing all following statements;
`break` can be used to break out of the loop unconditionally.


Iterate Through Arrays
----------------------

Iterating through an [array] yields cloned _copies_ of each element.

```rust
let a = [1, 3, 5, 7, 9, 42];

// Loop through the array
for x in a {
    if x > 10 { continue; }         // skip to the next iteration

    print(x);

    if x == 42 { break; }           // break out of for loop
}
```

Iterate Through Strings
-----------------------

Iterating through a [string] yields individual [characters].

The `chars` method also allow iterating through characters in a [string], optionally accepting the
character position to start from (counting from the end if negative), as well as the number of
characters to iterate (defaults to all).

`char` also accepts a [range] which can be created via the `..` (exclusive) and `..=` (inclusive) operators.

```rust
let s = "hello, world!";

// Iterate through all the characters.
for ch in s {
    print(ch);
}

// Iterate starting from the 3rd character and stopping at the 7th.
for ch in s.chars(2, 5) {
    if ch > 'z' { continue; }       // skip to the next iteration

    print(ch);

    if x == '@' { break; }          // break out of for loop
}

// Iterate starting from the 3rd character and stopping at the end.
for ch in s.chars(2..s.len) {
    if ch > 'z' { continue; }       // skip to the next iteration

    print(ch);

    if x == '@' { break; }          // break out of for loop
}
```


Iterate Through Numeric Ranges
------------------------------

[Ranges] are created via the `..` (exclusive) and `..=` (inclusive) operators.

The `range` function similarly creates exclusive [ranges], plus allowing optional step values.

```rust
// Iterate starting from 0 and stopping at 49
// The step is assumed to be 1 when omitted for integers
for x in 0..50 {
    if x > 10 { continue; }         // skip to the next iteration

    print(x);

    if x == 42 { break; }           // break out of for loop
}

// The 'range' function is just the same
for x in range(0, 50) {
    if x > 10 { continue; }         // skip to the next iteration

    print(x);

    if x == 42 { break; }           // break out of for loop
}

// The 'range' function also takes a step
for x in range(0, 50, 3) {          // step by 3
    if x > 10 { continue; }         // skip to the next iteration

    print(x);

    if x == 42 { break; }           // break out of for loop
}

// The 'range' function can also step backwards
for x in range(50..0, -3) {         // step down by -3
    if x < 10 { continue; }         // skip to the next iteration

    print(x);

    if x == 42 { break; }           // break out of for loop
}

// It works also for floating-point numbers
for x in range(5.0, 0.0, -2.0) {    // step down by -2.0
    if x < 10 { continue; }         // skip to the next iteration

    print(x);

    if x == 4.2 { break; }          // break out of for loop
}
```

Iterate Through Bit-Fields
--------------------------

The `bits` function allows iterating through an integer as a [bit-field].

`bits` optionally accepts the bit number to start from (counting from the most-significant-bit if
negative), as well as the number of bits to iterate (defaults all).

`bits` also accepts a [range] which can be created via the `..` (exclusive) and `..=` (inclusive) operators.

```js , no_run
let x = 0b_1001110010_1101100010_1100010100;
let num_on = 0;

// Iterate through all the bits
for bit in x.bits() {
    if bit { num_on += 1; }
}

print(`There are ${num_on} bits turned on!`);

const START = 3;

// Iterate through all the bits from 3 through 12
for (bit, index) in x.bits(START, 10) {
    print(`Bit #${index} is ${if bit { "ON" } else { "OFF" }}!`);

    if index >= 7 { break; }        // break out of for loop
}

// Iterate through all the bits from 3 through 12
for (bit, index) in x.bits(3..=12) {
    print(`Bit #${index} is ${if bit { "ON" } else { "OFF" }}!`);

    if index >= 7 { break; }        // break out of for loop
}
```

Iterate Through Object Maps
---------------------------

Two methods, `keys` and `values`, return [arrays] containing cloned _copies_
of all property names and values of an [object map], respectively.

These [arrays] can be iterated.

```rust
let map = #{a:1, b:3, c:5, d:7, e:9};

// Property names are returned in unsorted, random order
for x in map.keys() {
    if x > 10 { continue; }         // skip to the next iteration

    print(x);

    if x == 42 { break; }           // break out of for loop
}

// Property values are returned in unsorted, random order
for val in map.values() {
    print(val);
}
```
