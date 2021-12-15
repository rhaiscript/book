Ranges
======

{{#include ../links.md}}

Numeric ranges can be constructed by the `..` (exclusive) or `..=` (inclusive) operators.

> _start_ `..` _end_
>
> _start_ `..=` _end_

An _exclusive_ range does not include the last (i.e. "end") value, while an _inclusive_ range does.

The Rust type of a range `std::ops::Range` (exclusive) or `std::ops::RangeInclusive` (inclusive).

[`type_of()`] a range returns `"range"` (exclusive) or `"range="` (inclusive).


Built-in Functions
-----------------

The following methods (mostly defined in the [`BasicIteratorPackage`][packages] but excluded if
using a [raw `Engine`]) operate on ranges:

| Function                           |   Parameters    | Description                                   |
| ---------------------------------- | :-------------: | --------------------------------------------- |
| `start` method and property        |                 | beginning of the range                        |
| `end` method and property          |                 | end of the range                              |
| `contains`                         | number to check | does this range contain the specified number? |
| `is_inclusive` method and property |                 | is the range inclusive?                       |
