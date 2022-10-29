Do Loop
=======

`do` loops have two opposite variants: `do` ... `while` and `do` ... `until`.

`continue` can be used to skip to the next iteration, by-passing all following statements;
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


Do Expression
-------------

`do` statements can also be used as _expressions_.

The `break` statement takes an optional expression that provides the return value.

The default return value of a `do` expression is `()`.

```js
let x = 0;

// 'do' can be used just like an expression
let result = do {
    if is_magic_number(x) {
        // if the 'do' loop breaks here, return a specific value
        break get_magic_result(x);
    }

    x += 1;

    // ... if the 'do' loop exits here, the return value is ()
} until x >= 100;

if result == () {
    print("Magic number not found!");
} else {
    print(`Magic result = ${result}!`);
}
```
