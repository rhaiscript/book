`do` Loop
=========

{{#include ../links.md}}

`do` loops have two opposite variants: `do` ... `while` and `do` ... `until`.

Like the `while` loop, `continue` can be used to skip to the next iteration, by-passing all following statements;
`break` can be used to break out of the loop unconditionally.

```rust
let x = 10;

do {
    x -= 1;
    if x < 6 { continue; }  // skip to the next iteration
    print(x);
    if x == 5 { break; }    // break out of do loop
} while x > 0;


do {
    x -= 1;
    if x < 6 { continue; }  // skip to the next iteration
    print(x);
    if x == 5 { break; }    // break out of do loop
} until x == 0;
```
