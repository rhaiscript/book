Numeric Operators
=================

{{#title Numeric Operators}}

Numeric operators generally follow C styles.


Unary Operators
---------------

| Operator | Description |
| :------: | ----------- |
|   `+`    | positive    |
|   `-`    | negative    |

```rust
let number = +42;

number = -5;

number = -5 - +5;

-(-42) == +42;      // two '-' equals '+'
                    // beware: '++' and '--' are reserved symbols
```

Binary Operators
----------------

|             Operator              | Description                                                      |    Result type     | Integer |     Floating-point     |      Decimal      |
| :-------------------------------: | ---------------------------------------------------------------- | :----------------: | :-----: | :--------------------: | :---------------: |
|             `+`, `+=`             | plus                                                             |      numeric       |   yes   |   yes, also integer    | yes, also integer |
|             `-`, `-=`             | minus                                                            |      numeric       |   yes   |   yes, also integer    | yes, also integer |
|             `*`, `*=`             | multiply                                                         |      numeric       |   yes   |   yes, also integer    | yes, also integer |
|             `/`, `/=`             | divide (integer division if acting on integer types)             |      numeric       |   yes   |   yes, also integer    | yes, also integer |
|             `%`, `%=`             | modulo (remainder)                                               |      numeric       |   yes   |   yes, also integer    | yes, also integer |
|            `**`, `**=`            | power/exponentiation                                             |      numeric       |   yes   | yes, also `FLOAT**INT` |      **no**       |
|            `<<`, `<<=`            | left bit-shift (if negative number of bits, shift right instead) |      numeric       |   yes   |         **no**         |      **no**       |
|            `>>`, `>>=`            | right bit-shift (if negative number of bits, shift left instead) |      numeric       |   yes   |         **no**         |      **no**       |
|             `&`, `&=`             | bit-wise _And_                                                   |      numeric       |   yes   |         **no**         |      **no**       |
| <code>\|</code>, <code>\|=</code> | bit-wise _Or_                                                    |      numeric       |   yes   |         **no**         |      **no**       |
|             `^`, `^=`             | bit-wise _Xor_                                                   |      numeric       |   yes   |         **no**         |      **no**       |
|               `==`                | equals to                                                        |       `bool`       |   yes   |   yes, also integer    | yes, also integer |
|               `!=`                | not equals to                                                    |       `bool`       |   yes   |   yes, also integer    | yes, also integer |
|                `>`                | greater than                                                     |       `bool`       |   yes   |   yes, also integer    | yes, also integer |
|               `>=`                | greater than or equals to                                        |       `bool`       |   yes   |   yes, also integer    | yes, also integer |
|                `<`                | less than                                                        |       `bool`       |   yes   |   yes, also integer    | yes, also integer |
|               `<=`                | less than or equals to                                           |       `bool`       |   yes   |   yes, also integer    | yes, also integer |
|               `..`                | exclusive range                                                  | [range](ranges.md) |   yes   |         **no**         |      **no**       |
|               `..=`               | inclusive range                                                  | [range](ranges.md) |   yes   |         **no**         |      **no**       |


Examples
--------

```rust
let x = (1 + 2) * (6 - 4) / 2;  // arithmetic, with parentheses

let reminder = 42 % 10;         // modulo

let power = 42 ** 2;            // power

let left_shifted = 42 << 3;     // left shift

let right_shifted = 42 >> 3;    // right shift

let bit_op = 42 | 99;           // bit masking
```


Floating-Point Interoperates with Integers
------------------------------------------

When one of the operands to a binary arithmetic operator is floating-point, it works with integer
for the other operand and the result is floating-point.

```rust
let x = 41.0 + 1;               // float + integer

type_of(x) == "f64";            // result is float

let x = 21 * 2.0;               // float * integer

type_of(x) == "f64";

(x == 42) == true;              // float == integer

(10 < x) == true;               // integer < float
```


Decimal Interoperates with Integers
-----------------------------------

When one of the operands to a binary arithmetic operator is decimal,
it works with integer for the other operand and the result is decimal.

```rust
let d = parse_decimal("2");

let x = d + 1;                  // decimal + integer

type_of(x) == "decimal";        // result is decimal

let x = 21 * d;                 // decimal * integer

type_of(x) == "decimal";

(x == 42) == true;              // decimal == integer

(10 < x) == true;               // integer < decimal
```


Unary Before Binary
-------------------

In Rhai, unary operators take [precedence] over binary operators.  This is especially important to
remember when handling operators such as `**` which in some languages bind tighter than the unary
`-` operator.

```rust
-2 + 2 == 0;

-2 - 2 == -4;

-2 * 2 == -4;

-2 / 2 == -1;

-2 % 2 == 0;

-2 ** 2 = 4;            // means: (-2) ** 2
                        // in some languages this means: -(2 ** 2)
```
