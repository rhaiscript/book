BLOB's
======

{{#include ../links.md}}

BLOB's (**B**inary **L**arge **OB**jects), used to hold packed arrays of bytes, have built-in support in Rhai.

A BLOB has no literal representation, but is created via the `blob` function.

All items stored in a BLOB are bytes (i.e. `u8`) and the BLOB can freely grow or shrink with bytes
added or removed.

The Rust type of a Rhai BLOB is `rhai::Blob` which is an alias to `Vec<u8>`.

[`type_of()`] a BLOB returns `"blob"`.

BLOB's are disabled via the [`no_index`] feature.

The maximum allowed size of a BLOB can be controlled via [`Engine::set_max_array_size`][options]
(see [maximum size of arrays]).


Element Access
--------------

### From beginning

Like [arrays], BLOB's are accessed with zero-based, non-negative integer indices:

> _blob_ `[` _index from 0 to (length&minus;1)_ `]`

### From end

A _negative_ index accesses an element in the BLOB counting from the _end_, with &minus;1 being the
_last_ element.

> _blob_ `[` _index from &minus;1 to &minus;length_ `]`

### Byte values

The value of a particular byte in a BLOB is mapped to an `INT` (which can be 64-bit or 32-bit
depending on the [`only_i32`] feature).  Only the lowest 8 bits are significant, all other bits are
ignored.


Built-in Functions
-----------------

The following methods (mostly defined in the [`BasicBlobPackage`][packages] but excluded if using a [raw `Engine`]) operate on BLOB's:

| Method                              | Parameter(s)                                                                                                                                                                                      | Description                                                                                                                                          |
| ----------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------- |
| `blob` (constructor function)       | 1) _(optional)_ initial length of the BLOB<br/>2) _(optional)_ initial byte value                                                                                                                 | creates a new BLOB, optionally of a particular length filled with an initial byte value (default = 0)                                                |
| `push`                              | byte to insert                                                                                                                                                                                    | inserts a byte at the end                                                                                                                            |
| `append`                            | BLOB to append                                                                                                                                                                                    | concatenates the second BLOB to the end of the first                                                                                                 |
| `+=` operator                       | 1) BLOB<br/>2) byte to insert (not another BLOB)                                                                                                                                                  | inserts a byte at the end                                                                                                                            |
| `+=` operator                       | 1) BLOB<br/>2) BLOB to append                                                                                                                                                                     | concatenates the second BLOB to the end of the first                                                                                                 |
| `+` operator                        | 1) first BLOB<br/>2) second BLOB                                                                                                                                                                  | concatenates the first BLOB with the second                                                                                                          |
| `==` operator                       | 1) first BLOB<br/>2) second BLOB                                                                                                                                                                  | are the two BLOB's the same?                                                                                                                         |
| `!=` operator                       | 1) first BLOB<br/>2) second BLOB                                                                                                                                                                  | are the two BLOB's different?                                                                                                                        |
| `insert`                            | 1) position, counting from end if < 0, end if ≥ length<br/>2) byte to insert                                                                                                                      | inserts a byte at a certain index                                                                                                                    |
| `pop`                               | _none_                                                                                                                                                                                            | removes the last byte and returns it (0 if empty)                                                                                                    |
| `shift`                             | _none_                                                                                                                                                                                            | removes the first byte and returns it (0 if empty)                                                                                                   |
| `extract`                           | 1) start position, counting from end if < 0, end if ≥ length<br/>2) _(optional)_ number of bytes to extract, none if ≤ 0                                                                          | extracts a portion of the BLOB into a new BLOB                                                                                                       |
| `remove`                            | index                                                                                                                                                                                             | removes a byte at a particular index and returns it (0 if the index is not valid)                                                                    |
| `reverse`                           | _none_                                                                                                                                                                                            | reverses the BLOB byte by byte                                                                                                                       |
| `len` method and property           | _none_                                                                                                                                                                                            | returns the number of bytes in the BLOB                                                                                                              |
| `pad`                               | 1) target length<br/>2) byte value to pad                                                                                                                                                         | pads the BLOB with a byte value to at least a specified length                                                                                       |
| `clear`                             | _none_                                                                                                                                                                                            | empties the BLOB                                                                                                                                     |
| `truncate`                          | target length                                                                                                                                                                                     | cuts off the BLOB at exactly a specified length (discarding all subsequent bytes)                                                                    |
| `chop`                              | target length                                                                                                                                                                                     | cuts off the head of the BLOB, leaving the tail at exactly a specified length                                                                        |
| `contains`                          | byte value to find                                                                                                                                                                                | does the BLOB contain a particular byte value?                                                                                                       |
| `split`                             | 1) BLOB<br/>2) position to split at, counting from end if < 0, end if ≥ length                                                                                                                    | splits the BLOB into two BLOB's, starting from a specified position                                                                                  |
| `drain`                             | 1) start position, counting from end if < 0, end if ≥ length<br/>2) number of bytes to remove, none if ≤ 0                                                                                        | removes a portion of the BLOB, returning the removed bytes                                                                                           |
| `retain`                            | 1) start position, counting from end if < 0, end if ≥ length<br/>2) number of bytes to retain, none if ≤ 0                                                                                        | retains a portion of the BLOB, removes all other bytes and returning them                                                                            |
| `splice`                            | 1) start position, counting from end if < 0, end if ≥ length<br/>2) number of bytes to remove, none if ≤ 0<br/>3) BLOB to insert                                                                  | replaces a portion of the BLOB with another (not necessarily of the same length as the replaced portion)                                             |
| `parse_le_int`                      | 1) start position, counting from end if < 0, end if ≥ length<br/>2) number of bytes to parse, 8 if >8 (4 under [`only_i32`]), none if ≤ 0                                                         | parses an integer at the particular offset in little-endian byte order (if not enough bytes, zeros are padded; extra bytes are ignored)              |
| `parse_be_int`                      | 1) start position, counting from end if < 0, end if ≥ length<br/>2) number of bytes to parse, 8 if >8 (4 under [`only_i32`]), none if ≤ 0                                                         | parses an integer at the particular offset in big-endian byte order (if not enough bytes, zeros are padded; extra bytes are ignored)                 |
| `parse_le_float` (not [`no_float`]) | 1) start position, counting from end if < 0, end if ≥ length<br/>2) number of bytes to parse, 8 if >8 (4 under [`f32_float`]), none if ≤ 0                                                        | parses a floating-point number at the particular offset in little-endian byte order (if not enough bytes, zeros are padded; extra bytes are ignored) |
| `parse_be_float` (not [`no_float`]) | 1) start position, counting from end if < 0, end if ≥ length<br/>2) number of bytes to parse, 8 if >8 (4 under [`f32_float`]), none if ≤ 0                                                        | parses a floating-point number at the particular offset in big-endian byte order (if not enough bytes, zeros are padded; extra bytes are ignored)    |
| `write_le`                          | 1) start position, counting from end if < 0, end if ≥ length<br/>2) number of bytes to write, 8 if >8 (4 under [`only_i32`] or [`f32_float`]), none if ≤ 0<br/>3) integer or floating-point value | writes a value at the particular offset in little-endian byte order (if not enough bytes, zeros are padded; extra bytes are ignored)                 |
| `write_be`                          | 1) start position, counting from end if < 0, end if ≥ length<br/>2) number of bytes to write, 8 if >8 (4 under [`only_i32`] or [`f32_float`]), none if ≤ 0<br/>3) integer or floating-point value | writes a value at the particular offset in big-endian byte order (if not enough bytes, zeros are padded; extra bytes are ignored)                    |
