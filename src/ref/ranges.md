Ranges
======


Syntax
------

Numeric ranges can be constructed by the `..` (exclusive) or `..=` (inclusive) operators.

### Exclusive range

> _start_ `..` _end_

An _exclusive_ range does not include the last (i.e. "end") value.

[`type_of()`](type-of.md) an exclusive range returns `"range"`.

### Inclusive range

> _start_ `..=` _end_

An _inclusive_ range includes the last (i.e. "end") value.

[`type_of()`](type-of.md) an inclusive range returns `"range="`.


Usage Scenarios
---------------

Ranges are commonly used in the following scenarios.

| Scenario                                     | Example                                 |
| -------------------------------------------- | --------------------------------------- |
| [`for`](for.md) statements                   | `for n in 0..100 { ... }`               |
| [`in`](operators.md) expressions             | `if n in 0..100 { ... }`                |
| [`switch`](switch.md) expressions            | `switch n { 0..100 => ... }`            |
| [Bit-fields](bit-fields.md) access           | `let x = n[2..6];`                      |
| Bits iteration                               | `for bit in n.bits(2..=9) { ... }`      |
| [Array](arrays.md) range-based APIs          | `array.extract(2..8)`                   |
| [BLOB](blobs.md) range-based APIs            | `blob.parse_le_int(4..8)`               |
| [String](strings-chars.md) range-based APIs  | `string.sub_string(4..=12)`             |
| [Characters](strings-chars.md) iteration     | `for ch in string.bits(4..=12) { ... }` |


Built-in Functions
------------------

The following methods operate on ranges.

| Function                           |  Parameter(s)   | Description                                   |
| ---------------------------------- | :-------------: | --------------------------------------------- |
| `start` method and property        |                 | beginning of the range                        |
| `end` method and property          |                 | end of the range                              |
| `contains`, `in` operator          | number to check | does this range contain the specified number? |
| `is_empty` method and property     |                 | returns `true` if the range contains no items |
| `is_inclusive` method and property |                 | is the range inclusive?                       |
| `is_exclusive` method and property |                 | is the range exclusive?                       |


TL;DR
-----

```admonish question "What happened to the _open-ended_ ranges?"

Rust has _open-ended_ ranges, such as `start..`, `..end` and `..=end`.  They are not available in Rhai.

They are not needed because Rhai can overload functions.

Typically, an API accepting ranges as parameters would have equivalent versions that accept a
starting position and a length (the standard `start + len` pair), as well as a versions that accept
only the starting position (the length assuming to the end).

In fact, usually all versions redirect to a call to one single version.

Therefore, there should always be a function that can do what open-ended ranges are intended for.

The left-open form (i.e. `..end` and `..=end`) is trivially replaced by using zero as the starting
position with a length that corresponds to the end position (for `..end`).

The right-open form (i.e. `start..`) is trivially replaced by the version taking a single starting position.

~~~rust
let x = [1, 2, 3, 4, 5];

x.extract(0..3);    // normal range argument
                    // copies 'x' from positions 0-2

x.extract(2);       // copies 'x' from position 2 onwards
                    // equivalent to '2..'

x.extract(0, 2);    // copies 'x' from beginning for 2 items
                    // equivalent to '..2'
~~~
```
