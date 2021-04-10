`for` Loop
==========

{{#include ../links.md}}

Iterating through a range or an [array], or any type with a registered [type iterator],
is provided by the `for` ... `in` loop.

Like C, `continue` can be used to skip to the next iteration, by-passing all following statements;
`break` can be used to break out of the loop unconditionally.

To loop through a number sequence (with or without steps), use the `range` function to
return a numeric iterator.


Iterate Through Strings
-----------------------

Iterating through a [string] yields characters.

```rust , no_run
let s = "hello, world!";

for ch in s {
    if ch > 'z' { continue; }   // skip to the next iteration

    print(ch);

    if x == '@' { break; }      // break out of for loop
}
```


Iterate Through Arrays
----------------------

Iterating through an [array] yields cloned _copies_ of each element.

```rust , no_run
let array = [1, 3, 5, 7, 9, 42];

for x in array {
    if x > 10 { continue; }     // skip to the next iteration

    print(x);

    if x == 42 { break; }       // break out of for loop
}
```


Iterate Through Numeric Ranges
-----------------------------

The `range` function allows iterating through a range of numbers
(not including the last number).

```rust , no_run
// Iterate starting from 0 and stopping at 49.
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
for x in range(50, 0, -3) {     // step by -3
    if x < 10 { continue; }     // skip to the next iteration

    print(x);

    if x == 42 { break; }       // break out of for loop
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
