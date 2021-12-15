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

```rust no_run
let number = -5;

number = -5 - +5;
```

Binary Operators
----------------

| Operator                          | Description                                          | Return value | Integer |     Floating-point     | [`Decimal`][rust_decimal] |
| --------------------------------- | ---------------------------------------------------- | :----------: | :-----: | :--------------------: | :-----------------------: |
| `+`, `+=`                         | plus                                                 |   numeric    |   yes   |  yes, also with `INT`  |   yes, also with `INT`    |
| `-` `-=`                          | minus                                                |   numeric    |   yes   |  yes, also with `INT`  |   yes, also with `INT`    |
| `*`, `*=`                         | multiply                                             |   numeric    |   yes   |  yes, also with `INT`  |   yes, also with `INT`    |
| `/`, `/=`                         | divide (integer division if acting on integer types) |   numeric    |   yes   |  yes, also with `INT`  |   yes, also with `INT`    |
| `%`, `%=`                         | modulo (remainder)                                   |   numeric    |   yes   |  yes, also with `INT`  |   yes, also with `INT`    |
| `**`, `**=`                       | power/exponentiation                                 |   numeric    |   yes   | yes, also `FLOAT**INT` |            no             |
| `&`, `&=`                         | bit-wise _And_                                       |   numeric    |   yes   |           no           |            no             |
| <code>\|</code>, <code>\|=</code> | bit-wise _Or_                                        |   numeric    |   yes   |           no           |            no             |
| `^`                               | bit-wise _Xor_                                       |   numeric    |   yes   |           no           |            no             |
| `<<`, `<<=`                       | left bit-shift                                       |   numeric    |   yes   |           no           |            no             |
| `>>`, `>>=`                       | right bit-shift                                      |   numeric    |   yes   |           no           |            no             |
| `==`                              | equals to                                            |    `bool`    |   yes   |  yes, also with `INT`  |   yes, also with `INT`    |
| `!=`                              | not equals to                                        |    `bool`    |   yes   |  yes, also with `INT`  |   yes, also with `INT`    |
| `>`                               | greater than                                         |    `bool`    |   yes   |  yes, also with `INT`  |   yes, also with `INT`    |
| `>=`                              | greater than or equals to                            |    `bool`    |   yes   |  yes, also with `INT`  |   yes, also with `INT`    |
| `<`                               | less than                                            |    `bool`    |   yes   |  yes, also with `INT`  |   yes, also with `INT`    |
| `<=`                              | less than or equals to                               |    `bool`    |   yes   |  yes, also with `INT`  |   yes, also with `INT`    |
| `..`                              | exclusive range                                      |   [range]    |   yes   |           no           |            no             |
| `..=`                             | inclusive range                                      |   [range]    |   yes   |           no           |            no             |

Note: when one of the operands to a binary operator is floating-point, it works with `INT` for the
other operand and the result is floating-point.

```rust no_run
let x = (1 + 2) * (6 - 4) / 2;  // arithmetic, with parentheses

let reminder = 42 % 10;         // modulo

let power = 42 ** 2;            // power

let left_shifted = 42 << 3;     // left shift

let right_shifted = 42 >> 3;    // right shift

let bit_op = 42 | 99;           // bit masking
```


Unary Before Binary
-------------------

In Rhai, unary operators take precedence over binary operators.  This is especially important to
remember when handling operators such as `**` which in some languages bind tighter than the unary
`-` operator.

```rust no_run
-2 + 2 == 0;

-2 - 2 == -4;

-2 * 2 == -4;

-2 / 2 == -1;

-2 % 2 == 0;

-2 ** 2 = 4;    // in some languages this means -(2 ** 2) == -4
```
