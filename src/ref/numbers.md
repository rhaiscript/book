Numbers
=======


Integers
--------

Integer numbers follow C-style format with support for decimal, binary (`0b`), octal (`0o`) and hex (`0x`) notations.

Integers can also be conveniently manipulated as [bit-fields](bit-fields.md).


Floating-Point Numbers
----------------------

Both decimal and scientific notations can be used to represent floating-point numbers.


Decimal Numbers
---------------

When rounding errors cannot be accepted, such as in financial calculations, use the decimal type,
which is a fixed-precision floating-point number with no rounding errors.


Number Literals
---------------

`_` separators can be added freely and are ignored within a number &ndash; except at the very
beginning or right after a decimal point (`.`).

| Sample             | Format                    |
| ------------------ | ------------------------- |
| `_123`             | _improper separator_      |
| `123_345`, `-42`   | decimal                   |
| `0o07_76`          | octal                     |
| `0xab_cd_ef`       | hex                       |
| `0b0101_1001`      | binary                    |
| `123._456`         | _improper separator_      |
| `123_456.78_9`     | normal floating-point     |
| `-42.`             | ending with decimal point |
| `123_456_.789e-10` | scientific notation       |
| `.456`             | _missing leading `0`_     |
| `123.456e_10`      | _improper separator_      |
| `123.e-10`         | _missing decimal `0`_     |


Floating-Point vs. Decimal
--------------------------

Decimal numbers represents a fixed-precision floating-point number which is popular with financial
calculations and other usage scenarios where round-off errors are not acceptable.

Decimal numbers take up more space (16 bytes each) than a standard floating-point number (4-8 bytes)
and is much slower in calculations due to the lack of CPU hardware support. Use it only when
necessary.
