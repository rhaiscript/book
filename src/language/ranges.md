Range
=====

{{#include ../links.md}}


Syntax
------

Numeric ranges can be constructed by the `..` (exclusive) or `..=` (inclusive) operators.

### Exclusive range

> _start_ `..` _end_

An _exclusive_ range does not include the last (i.e. "end") value.

The Rust type of an exclusive range is `std::ops::Range<INT>`.

[`type_of()`] an exclusive range returns `"range"`.

### Inclusive range

> _start_ `..=` _end_

An _inclusive_ range includes the last (i.e. "end") value.

The Rust type of an inclusive range is `std::ops::RangeInclusive<INT>`.

[`type_of()`] an inclusive range returns `"range="`.


Usage Scenarios
---------------

Ranges are commonly used in the following scenarios.

| Scenario                   | Example                                 |
| -------------------------- | --------------------------------------- |
| [`for`] statements         | `for n in 0..100 { ... }`               |
| [`in`] expressions         | `if n in 0..100 { ... }`                |
| [`switch`] expressions     | `switch n { 0..100 => ... }`            |
| [Bit-fields] access        | `let x = n[2..6];`                      |
| [Array] range-based API's  | `array.extract(2..8)`                   |
| [String] range-based API's | `string.sub_string(4..=12)`             |
| [BLOB] range-based API's   | `blob.parse_le_int(4..8)`               |
| Bits iteration             | `for bit in n.bits(2..=9) { ... }`      |
| [Characters] iteration     | `for ch in string.bits(4..=12) { ... }` |


Built-in Functions
-----------------

The following methods (mostly defined in the [`BasicIteratorPackage`][packages] but excluded if
using a [raw `Engine`]) operate on ranges.

| Function                           |  Parameter(s)   | Description                                   |
| ---------------------------------- | :-------------: | --------------------------------------------- |
| `start` method and property        |                 | beginning of the range                        |
| `end` method and property          |                 | end of the range                              |
| `contains`                         | number to check | does this range contain the specified number? |
| `is_inclusive` method and property |                 | is the range inclusive?                       |
| `is_exclusive` method and property |                 | is the range exclusive?                       |
