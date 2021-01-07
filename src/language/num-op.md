Numeric Operators
=================

{{#include ../links.md}}

Numeric operators generally follow C styles.

Unary Operators
---------------

| Operator | Description |
| -------- | ----------- |
| `+`      | positive    |
| `-`      | negative    |

```rust
let number = -5;

number = -5 - +5;
```

Binary Operators
----------------

| Operator        | Description                                          | Integers only |
| --------------- | ---------------------------------------------------- | :-----------: |
| `+`             | plus                                                 |               |
| `-`             | minus                                                |               |
| `*`             | multiply                                             |               |
| `/`             | divide (integer division if acting on integer types) |               |
| `%`             | modulo (remainder)                                   |               |
| `~`             | power                                                |               |
| `&`             | bit-wise _And_                                       |      Yes      |
| <code>\|</code> | bit-wise _Or_                                        |      Yes      |
| `^`             | bit-wise _Xor_                                       |      Yes      |
| `<<`            | left bit-shift                                       |      Yes      |
| `>>`            | right bit-shift                                      |      Yes      |

```rust
let x = (1 + 2) * (6 - 4) / 2;  // arithmetic, with parentheses

let reminder = 42 % 10;         // modulo

let power = 42 ~ 2;             // power (i64 and f64 only)

let left_shifted = 42 << 3;     // left shift

let right_shifted = 42 >> 3;    // right shift

let bit_op = 42 | 99;           // bit masking
```
