`for` Loop
==========

{{#include ../links.md}}

Iterating through a range or an [array], or any type with a registered [type iterator],
is provided by the `for` ... `in` loop.

Like C, `continue` can be used to skip to the next iteration, by-passing all following statements;
`break` can be used to break out of the loop unconditionally.

To loop through a number sequence (with or without steps), use the `range` function to
return a numeric iterator.


Iterate Through Arrays
----------------------

Iterating through an [array] yields cloned _copies_ of each element.

```rust , no_run
let a = [1, 3, 5, 7, 9, 42];

for x in a {
    if x > 10 { continue; }     // skip to the next iteration

    print(x);

    if x == 42 { break; }       // break out of for loop
}
```


Iterate Through Strings
-----------------------

The `chars` method allows iterating through a [string], yielding characters.

`chars` optionally accepts the character to start from (counting from the end if negative), as well
as the number of characters to iterate (defaults all).

```rust , no_run
let s = "hello, world!";

// Iterate through all the characters.
for ch in s.chars() {
    print(ch);
}

// Iterate starting from the 3rd character and stopping at the 7th.
for ch in s.chars(2, 5) {
    if ch > 'z' { continue; }   // skip to the next iteration

    print(ch);

    if x == '@' { break; }      // break out of for loop
}
```


Iterate Through Numeric Ranges
-----------------------------

The `range` function allows iterating through a range of numbers
(not including the last number).

```rust , no_run
// Iterate starting from 0 and stopping at 49.
// The step is assumed to be 1 when omitted for integers.
for x in range(0, 50) {
    if x > 10 { continue; }     // skip to the next iteration

    print(x);

    if x == 42 { break; }       // break out of for loop
}

// The 'range' function also takes a step.
for x in range(0, 50, 3) {      // step by 3
    if x > 10 { continue; }     // skip to the next iteration

    print(x);

    if x == 42 { break; }       // break out of for loop
}

// The 'range' function can also step backwards.
for x in range(50, 0, -3) {     // step down by -3
    if x < 10 { continue; }     // skip to the next iteration

    print(x);

    if x == 42 { break; }       // break out of for loop
}

// It works also for floating-point numbers.
for x in range(5.0,0.0,-2.0) {  // step down by -2.0
    if x < 10 { continue; }     // skip to the next iteration

    print(x);

    if x == 4.2 { break; }      // break out of for loop
}
```


Iterate Through Bit-Fields
--------------------------

The `bits` function allows iterating through an integer as a [bit-field].

`bits` optionally accepts the bit number to start from (counting from the most-significant-bit if
negative), as well as the number of bits to iterate (defaults all).


```js , no_run
let x = 0b_1001110010_1101100010_1100010100;
let num_on = 0;

// Iterate through all the bits
for bit in x.bits() {
    if bit { num_on += 1; }
}

print(`There are ${num_on} bits turned on!`);

const START = 3;
let index = START;

// Iterate through all the bits from 3 through 12
for bit in x.bits(START, 10) {
    print(`Bit #${index} is ${if bit { "ON" } else { "OFF" }}!`);

    if index >= 32 { break; }   // break out of for loop

    index += 1;
}
```


Iterate Through Object Maps
--------------------------

Two methods, `keys` and `values`, return [arrays] containing cloned _copies_
of all property names and values of an [object map], respectively.

These [arrays] can be iterated.

```rust , no_run
let map = #{a:1, b:3, c:5, d:7, e:9};

// Property names are returned in unsorted, random order
for x in map.keys() {
    if x > 10 { continue; }     // skip to the next iteration

    print(x);

    if x == 42 { break; }       // break out of for loop
}

// Property values are returned in unsorted, random order
for val in map.values() {
    print(val);
}
```
