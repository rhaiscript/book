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
| `**`            | power/exponentiation                                 |               |
| `&`             | bit-wise _And_                                       |      Yes      |
| <code>\|</code> | bit-wise _Or_                                        |      Yes      |
| `^`             | bit-wise _Xor_                                       |      Yes      |
| `<<`            | left bit-shift                                       |      Yes      |
| `>>`            | right bit-shift                                      |      Yes      |

```rust
let x = (1 + 2) * (6 - 4) / 2;  // arithmetic, with parentheses

let reminder = 42 % 10;         // modulo

let power = 42 ** 2;            // power (i64 and f64 only)

let left_shifted = 42 << 3;     // left shift

let right_shifted = 42 >> 3;    // right shift

let bit_op = 42 | 99;           // bit masking
```


Unary Before Binary
-------------------

In Rhai, unary operators take precedence over binary operators.  This is especially important to
remember when handling operators such as `**` which in some languages bind tighter than the unary
`-` operator.

```rust
-2 + 2 == 0;

-2 - 2 == -4;

-2 * 2 == -4;

-2 / 2 == -1;

-2 % 2 == 0;

-2 ** 2 = 4;    // in some languages this means -(2 ** 2) == -4
```
