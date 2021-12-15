Ranges
======

{{#include ../links.md}}

Numeric ranges can be constructed by the `..` (exclusive) or `..=` (inclusive) operators.

> _start_ `..` _end_ &nbsp;&nbsp;&nbsp;&nbsp; (exclusive)
>
> _start_ `..=` _end_ &nbsp;&nbsp;&nbsp;&nbsp; (inclusive)

An _exclusive_ range does not include the last (i.e. "end") value, while an _inclusive_ range does.

The Rust type of a range `std::ops::Range` (exclusive) or `std::ops::RangeInclusive` (inclusive).

[`type_of()`] a range returns `"range"` (exclusive) or `"range="` (inclusive).


Usage Scenarios
---------------

Ranges are commonly used in these scenarios:

| Scenario                                               | Example                                 |
| ------------------------------------------------------ | --------------------------------------- |
| [`for`]({{rootUrl}}/language/for.md) statements        | `for n in 0..100 { ... }`               |
| [`in`]({{rootUrl}}/language/in.md) expressions         | `if n in 0..100 { ... }`                |
| [`switch`]({{rootUrl}}/language/switch.md) expressions | `switch n { 0..100 => ... }`            |
| [Bit-fields] access                                    | `let x = n[2..6];`                      |
| [Array] range-based API's                              | `array.extract(2..8)`                   |
| [String] range-based API's                             | `string.sub_string(4..=12)`             |
| [BLOB] range-based API's                               | `blob.parse_le_int(4..8)`               |
| Bits iteration                                         | `for bit in n.bits(2..=9) { ... }`      |
| [Characters] iteration                                 | `for ch in string.bits(4..=12) { ... }` |


Built-in Functions
-----------------

The following methods (mostly defined in the [`BasicIteratorPackage`][packages] but excluded if
using a [raw `Engine`]) operate on ranges:

| Function                           |   Parameters    | Description                                    |
| ---------------------------------- | :-------------: | ---------------------------------------------- |
| `start` method and property        |                 | beginning of the range                         |
| `end` method and property          |                 | end of the range                               |
| `contains`                         | number to check | does this range contain the specified number?  |
| `is_empty` method and property     |                 | is the range empty (i.e. contains no numbers)? |
| `is_inclusive` method and property |                 | is the range inclusive?                        |
| `is_exclusive` method and property |                 | is the range exclusive?                        |
