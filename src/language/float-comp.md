Floating-point Comparison
=========================

{{#include ../links.md}}


0.1 + 0.2 ≠ 0.3
---------------

```admonish info.side.wide "See Also"

See also [_this article_](https://randomascii.wordpress.com/2012/02/25/comparing-floating-point-numbers-2012-edition/)
for a detailed discussion on comparing floating-point numbers.
```

Comparing floating-point numbers are tricky because they often fail miserably and unexpectedly.

Take the infamous `0.1 + 0.2 ≠ 0.3` example from most languages:

```js
// JavaScript example

let x = 0.3;            // 0.3
let y = 0.1 + 0.2;      // 0.3?

console.log(x == y);    // false!

console.log(x);         // 0.3
console.log(y);         // 0.30000000000000004
```


Epsilon-Based Implementation
----------------------------

```admonish warning.small "Warning"

If the [`unchecked`] feature is enabled, then all floating-point comparisons are done as per IEEE 754 standard.
```

The IEEE 754 standard for floating-point arithmetic provides an
[_epsilon_](https://en.wikipedia.org/wiki/Machine_epsilon) value that can be used to compare floating-point numbers.

With `max_diff = max(|x|, |y|) × epsilon`, Rhai compares floating-point numbers based on the following:

| operator |   operands    | algorithm                   |
| :------: | :-----------: | --------------------------- |
| `x == y` |   has zero    | `\|x - y\|` < _epsilon_     |
| `x == y` | both non-zero | `\|x - y\|` < _epsilon_     |
| `x != y` |   has zero    | `\|x - y\|` ≥ _epsilon_     |
| `x != y` | both non-zero | `\|x - y\|` ≥ `max_diff`    |
| `x > y`  |   has zero    | `x - y` ≥ _epsilon_         |
| `x > y`  | both non-zero | `x - y` ≥ `max_diff`        |
| `x >= y` |   has zero    | `x - y` ≥ &ndash;_epsilon_  |
| `x >= y` | both non-zero | `x - y` ≥ &ndash;`max_diff` |
| `x < y`  |   has zero    | `y - x` ≥ _epsilon_         |
| `x < y`  | both non-zero | `y - x` ≥ `max_diff`        |
| `x <= y` |   has zero    | `y - x` ≥ &ndash;_epsilon_  |
| `x <= y` | both non-zero | `y - x` ≥ &ndash;`max_diff` |


Casual Scripting Usage
----------------------

For most scripts that casually compare floating-point numbers, the default behavior is sufficient to
cover all bases:

```rust
let x = 0.3;            // 0.3
let y = 0.1 + 0.2;      // 0.3?

x == y == true;         // 0.1 + 0.2 = 0.3. Just Works!

let x = 1e-16;          // 0.0000000000000001
x > 0 == false;         // comparing with zero Just Works when value is very small
x == 0 == true;

let x = 1e-15;          // 0.000000000000001
x > 0 == true;          // comparing with zero when value above epsilon
x == 0 == false;

let x = 1e-28;          // 0.0000000000000000000000000001
let y = 1e-29;          // 0.00000000000000000000000000001, 10x smaller than 'x'
x > y == true;          // comparing very small numbers actually wide apart!

let x = 0.00000000000000000001;
let y = 0.000000000000000000010000000000000001;
x < y == false;         // difference is small enough for 'x' == 'y'

let y = 0.00000000000000000001000000000000001;
x < y = true;           // difference is not small enough for 'x' == 'y'
```
