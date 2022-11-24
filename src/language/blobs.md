BLOB's
======

{{#include ../links.md}}

```admonish tip.side "Safety"

Always limit the [maximum size of arrays].
```

BLOB's (**B**inary **L**arge **OB**jects), used to hold packed arrays of bytes, have built-in support in Rhai.

A BLOB has no literal representation, but is created via the `blob` function, or simply returned as
the result of a function call (e.g. `generate_thumbnail_image` that generates a thumbnail version of
a large image as a BLOB).

All items stored in a BLOB are bytes (i.e. `u8`) and the BLOB can freely grow or shrink with bytes
added or removed.

The Rust type of a Rhai BLOB is `rhai::Blob` which is an alias to `Vec<u8>`.

[`type_of()`] a BLOB returns `"blob"`.

BLOB's are disabled via the [`no_index`] feature.


Element Access Syntax
---------------------

### From beginning

Like [arrays], BLOB's are accessed with zero-based, non-negative integer indices:

> _blob_ `[` _index position from 0 to (length−1)_ `]`

### From end

A _negative_ position accesses an element in the BLOB counting from the _end_, with −1 being the
_last_ element.

> _blob_ `[` _index position from −1 to −length_ `]`

```admonish info.small "Byte values"

The value of a particular byte in a BLOB is mapped to an `INT` (which can be 64-bit or 32-bit
depending on the [`only_i32`] feature).

Only the lowest 8 bits are significant, all other bits are ignored.
```


Create a BLOB
-------------

The function `blob` allows creating an empty BLOB, optionally filling it to a required size with a
particular value (default zero).

```rust
let x = blob();             // empty BLOB

let x = blob(10);           // BLOB with ten zeros

let x = blob(50, 42);       // BLOB with 50x 42's
```

```admonish tip "Tip: Initialize with byte stream"

To quickly initialize a BLOB with a particular byte stream, the `write_be` method can be used to
write eight bytes at a time (four under [`only_i32`]) in big-endian byte order.

If fewer than eight bytes are needed, remember to right-pad the number as big-endian byte order is used.

~~~rust
let buf = blob(12, 0);      // BLOB with 12x zeros

// Write eight bytes at a time, in big-endian order
buf.write_be(0, 8, 0xab_cd_ef_12_34_56_78_90);
buf.write_be(8, 8, 0x0a_0b_0c_0d_00_00_00_00);
                            //   ^^^^^^^^^^^ remember to pad unused bytes

print(buf);                 // prints "[abcdef1234567890 0a0b0c0d]"

buf[3] == 0x12;
buf[10] == 0x0c;

// Under 'only_i32', write four bytes at a time:
buf.write_be(0, 4, 0xab_cd_ef_12);
buf.write_be(4, 4, 0x34_56_78_90);
buf.write_be(8, 4, 0x0a_0b_0c_0d);
~~~
```


Writing ASCII Bytes
-------------------

```admonish warning.side "Non-ASCII"

Non-ASCII characters (i.e. characters not within 1-127) are ignored.
```

For many embedded applications, it is necessary to encode an ASCII [string] as a byte stream.

Use the `write_ascii` method to write ASCII [strings] into any specific [range] within a BLOB.

The following is an example of a building a 16-byte command to send to an embedded device.

```rust
// Assume the following 16-byte command for an embedded device:
// ┌─────────┬───────────────┬──────────────────────────────────┬───────┐
// │    0    │       1       │              2-13                │ 14-15 │
// ├─────────┼───────────────┼──────────────────────────────────┼───────┤
// │ command │ string length │ ASCII string, max. 12 characters │  CRC  │
// └─────────┴───────────────┴──────────────────────────────────┴───────┘

let buf = blob(16, 0);      // initialize command buffer

let text = "foo & bar";     // text string to send to device

buf[0] = 0x42;              // command code
buf[1] = s.len();           // length of string

buf.write_ascii(2..14, text);   // write the string

let crc = buf.calc_crc();   // calculate CRC

buf.write_le(14, 2, crc);   // write CRC

print(buf);                 // prints "[4209666f6f202620 626172000000abcd]"
                            //          ^^ command code              ^^^^ CRC
                            //            ^^ string length
                            //              ^^^^^^^^^^^^^^^^^^^ foo & bar

device.send(buf);           // send command to device
```

```admonish question.small "What if I need UTF-8?"

The `write_utf8` function writes a string in UTF-8 encoding.

UTF-8, however, is not very common for embedded applications.
```


Built-in Functions
------------------

The following functions (mostly defined in the [`BasicBlobPackage`][built-in packages] but excluded
when using a [raw `Engine`]) operate on BLOB's.

| Functions                                               | Parameter(s)                                                                                                                                                                                                        | Description                                                                                                                                          |
| ------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------- |
| `blob` constructor function                             | <ol><li>_(optional)_ initial length of the BLOB</li><li>_(optional)_ initial byte value</li></ol>                                                                                                                   | creates a new BLOB, optionally of a particular length filled with an initial byte value (default = 0)                                                |
| `to_array`                                              | _none_                                                                                                                                                                                                              | converts the BLOB into an [array] of integers                                                                                                        |
| `as_string`                                             | _none_                                                                                                                                                                                                              | converts the BLOB into a [string] (the byte stream is interpreted as UTF-8)                                                                          |
| `get`                                                   | position, counting from end if < 0                                                                                                                                                                                  | gets a copy of the byte at a certain position (0 if the position is not valid)                                                                       |
| `set`                                                   | <ol><li>position, counting from end if < 0</li><li>new byte value</li></ol>                                                                                                                                         | sets a certain position to a new value (no effect if the position is not valid)                                                                      |
| `push`, `append`, `+=` operator                         | <ol><li>BLOB</li><li>byte to append</li></ol>                                                                                                                                                                       | appends a byte to the end                                                                                                                            |
| `append`, `+=` operator                                 | <ol><li>BLOB</li><li>BLOB to append</li></ol>                                                                                                                                                                       | concatenates the second BLOB to the end of the first                                                                                                 |
| `append`, `+=` operator                                 | <ol><li>BLOB</li><li>[string]/[character] to append</li></ol>                                                                                                                                                       | concatenates a [string] or [character] (as UTF-8 encoded byte-stream) to the end of the BLOB                                                         |
| `+` operator                                            | <ol><li>first BLOB</li><li>[string] to append</li></ol>                                                                                                                                                             | creates a new [string] by concatenating the BLOB (as UTF-8 encoded byte-stream) with the the [string]                                                |
| `+` operator                                            | <ol><li>[string]</li><li>BLOB to append</li></ol>                                                                                                                                                                   | creates a new [string] by concatenating the BLOB (as UTF-8 encoded byte-stream) to the end of the [string]                                           |
| `+` operator                                            | <ol><li>first BLOB</li><li>second BLOB</li></ol>                                                                                                                                                                    | concatenates the first BLOB with the second                                                                                                          |
| `==` operator                                           | <ol><li>first BLOB</li><li>second BLOB</li></ol>                                                                                                                                                                    | are two BLOB's the same?                                                                                                                             |
| `!=` operator                                           | <ol><li>first BLOB</li><li>second BLOB</li></ol>                                                                                                                                                                    | are two BLOB's different?                                                                                                                            |
| `insert`                                                | <ol><li>position, counting from end if < 0, end if ≥ length</li><li>byte to insert</li></ol>                                                                                                                        | inserts a byte at a certain position                                                                                                                 |
| `pop`                                                   | _none_                                                                                                                                                                                                              | removes the last byte and returns it (0 if empty)                                                                                                    |
| `shift`                                                 | _none_                                                                                                                                                                                                              | removes the first byte and returns it (0 if empty)                                                                                                   |
| `extract`                                               | <ol><li>start position, counting from end if < 0, end if ≥ length</li><li>_(optional)_ number of bytes to extract, none if ≤ 0</li></ol>                                                                            | extracts a portion of the BLOB into a new BLOB                                                                                                       |
| `extract`                                               | [range] of bytes to extract, from beginning if ≤ 0, to end if ≥ length                                                                                                                                              | extracts a portion of the BLOB into a new BLOB                                                                                                       |
| `remove`                                                | position, counting from end if < 0                                                                                                                                                                                  | removes a byte at a particular position and returns it (0 if the position is not valid)                                                              |
| `reverse`                                               | _none_                                                                                                                                                                                                              | reverses the BLOB byte by byte                                                                                                                       |
| `len` method and property                               | _none_                                                                                                                                                                                                              | returns the number of bytes in the BLOB                                                                                                              |
| `is_empty` method and property                          | _none_                                                                                                                                                                                                              | returns `true` if the BLOB is empty                                                                                                                  |
| `pad`                                                   | <ol><li>target length</li><li>byte value to pad</li></ol>                                                                                                                                                           | pads the BLOB with a byte value to at least a specified length                                                                                       |
| `clear`                                                 | _none_                                                                                                                                                                                                              | empties the BLOB                                                                                                                                     |
| `truncate`                                              | target length                                                                                                                                                                                                       | cuts off the BLOB at exactly a specified length (discarding all subsequent bytes)                                                                    |
| `chop`                                                  | target length                                                                                                                                                                                                       | cuts off the head of the BLOB, leaving the tail at exactly a specified length                                                                        |
| `contains`, [`in`] operator                             | byte value to find                                                                                                                                                                                                  | does the BLOB contain a particular byte value?                                                                                                       |
| `split`                                                 | <ol><li>BLOB</li><li>position to split at, counting from end if < 0, end if ≥ length</li></ol>                                                                                                                      | splits the BLOB into two BLOB's, starting from a specified position                                                                                  |
| `drain`                                                 | <ol><li>start position, counting from end if < 0, end if ≥ length</li><li>number of bytes to remove, none if ≤ 0</li></ol>                                                                                          | removes a portion of the BLOB, returning the removed bytes as a new BLOB                                                                             |
| `drain`                                                 | [range] of bytes to remove, from beginning if ≤ 0, to end if ≥ length                                                                                                                                               | removes a portion of the BLOB, returning the removed bytes as a new BLOB                                                                             |
| `retain`                                                | <ol><li>start position, counting from end if < 0, end if ≥ length</li><li>number of bytes to retain, none if ≤ 0</li></ol>                                                                                          | retains a portion of the BLOB, removes all other bytes and returning them as a new BLOB                                                              |
| `retain`                                                | [range] of bytes to retain, from beginning if ≤ 0, to end if ≥ length                                                                                                                                               | retains a portion of the BLOB, removes all other bytes and returning them as a new BLOB                                                              |
| `splice`                                                | <ol><li>start position, counting from end if < 0, end if ≥ length</li><li>number of bytes to remove, none if ≤ 0</li><li>BLOB to insert</li></ol>                                                                   | replaces a portion of the BLOB with another (not necessarily of the same length as the replaced portion)                                             |
| `splice`                                                | <ol><li>[range] of bytes to remove, from beginning if ≤ 0, to end if ≥ length</li><li>BLOB to insert                                                                                                                | replaces a portion of the BLOB with another (not necessarily of the same length as the replaced portion)                                             |
| `parse_le_int`                                          | <ol><li>start position, counting from end if < 0, end if ≥ length</li><li>number of bytes to parse, 8 if > 8 (4 under [`only_i32`]), none if ≤ 0</li></ol>                                                          | parses an integer at the particular offset in little-endian byte order (if not enough bytes, zeros are padded; extra bytes are ignored)              |
| `parse_le_int`                                          | [range] of bytes to parse, from beginning if ≤ 0, to end if ≥ length (up to 8 bytes, 4 under [`only_i32`])                                                                                                          | parses an integer at the particular offset in little-endian byte order (if not enough bytes, zeros are padded; extra bytes are ignored)              |
| `parse_be_int`                                          | <ol><li>start position, counting from end if < 0, end if ≥ length</li><li>number of bytes to parse, 8 if > 8 (4 under [`only_i32`]), none if ≤ 0</li></ol>                                                          | parses an integer at the particular offset in big-endian byte order (if not enough bytes, zeros are padded; extra bytes are ignored)                 |
| `parse_be_int`                                          | [range] of bytes to parse, from beginning if ≤ 0, to end if ≥ length (up to 8 bytes, 4 under [`only_i32`])                                                                                                          | parses an integer at the particular offset in big-endian byte order (if not enough bytes, zeros are padded; extra bytes are ignored)                 |
| `parse_le_float`<br/>(not available under [`no_float`]) | <ol><li>start position, counting from end if < 0, end if ≥ length</li><li>number of bytes to parse, 8 if > 8 (4 under [`f32_float`]), none if ≤ 0</li></ol>                                                         | parses a floating-point number at the particular offset in little-endian byte order (if not enough bytes, zeros are padded; extra bytes are ignored) |
| `parse_le_float`<br/>(not available under [`no_float`]) | [range] of bytes to parse, from beginning if ≤ 0, to end if ≥ length (up to 8 bytes, 4 under [`f32_float`])                                                                                                         | parses a floating-point number at the particular offset in little-endian byte order (if not enough bytes, zeros are padded; extra bytes are ignored) |
| `parse_be_float`<br/>(not available under [`no_float`]) | <ol><li>start position, counting from end if < 0, end if ≥ length</li><li>number of bytes to parse, 8 if > 8 (4 under [`f32_float`]), none if ≤ 0</li></ol>                                                         | parses a floating-point number at the particular offset in big-endian byte order (if not enough bytes, zeros are padded; extra bytes are ignored)    |
| `parse_be_float`<br/>(not available under [`no_float`]) | [range] of bytes to parse, from beginning if ≤ 0, to end if ≥ length (up to 8 bytes, 4 under [`f32_float`])                                                                                                         | parses a floating-point number at the particular offset in big-endian byte order (if not enough bytes, zeros are padded; extra bytes are ignored)    |
| `write_le`                                              | <ol><li>start position, counting from end if < 0, end if ≥ length</li><li>number of bytes to write, 8 if > 8 (4 under [`only_i32`] or [`f32_float`]), none if ≤ 0</li><li>integer or floating-point value</li></ol> | writes a value at the particular offset in little-endian byte order (if not enough bytes, zeros are padded; extra bytes are ignored)                 |
| `write_le`                                              | <ol><li>[range] of bytes to write, from beginning if ≤ 0, to end if ≥ length (up to 8 bytes, 4 under [`only_i32`] or [`f32_float`])</li><li>integer or floating-point value</li></ol>                               | writes a value at the particular offset in little-endian byte order (if not enough bytes, zeros are padded; extra bytes are ignored)                 |
| `write_be`                                              | <ol><li>start position, counting from end if < 0, end if ≥ length</li><li>number of bytes to write, 8 if > 8 (4 under [`only_i32`] or [`f32_float`]), none if ≤ 0</li><li>integer or floating-point value</li></ol> | writes a value at the particular offset in big-endian byte order (if not enough bytes, zeros are padded; extra bytes are ignored)                    |
| `write_be`                                              | <ol><li>[range] of bytes to write, from beginning if ≤ 0, to end if ≥ length (up to 8 bytes, 4 under [`only_i32`] or [`f32_float`])</li><li>integer or floating-point value</li></ol>                               | writes a value at the particular offset in big-endian byte order (if not enough bytes, zeros are padded; extra bytes are ignored)                    |
| `write_utf8`                                            | <ol><li>start position, counting from end if < 0, end if ≥ length</li><li>number of bytes to write, none if ≤ 0, to end if ≥ length</li><li>[string] to write</li></ol>                                             | writes a [string] to the particular offset in UTF-8 encoding                                                                                         |
| `write_utf8`                                            | <ol><li>[range] of bytes to write, from beginning if ≤ 0, to end if ≥ length, to end if ≥ length</li><li>[string] to write</li></ol>                                                                                | writes a [string] to the particular offset in UTF-8 encoding                                                                                         |
| `write_ascii`                                           | <ol><li>start position, counting from end if < 0, end if ≥ length</li><li>number of [characters] to write, none if ≤ 0, to end if ≥ length</li><li>[string] to write</li></ol>                                      | writes a [string] to the particular offset in 7-bit ASCII encoding (non-ASCII [characters] are skipped)                                              |
| `write_ascii`                                           | <ol><li>[range] of bytes to write, from beginning if ≤ 0, to end if ≥ length, to end if ≥ length</li><li>[string] to write</li></ol>                                                                                | writes a [string] to the particular offset in 7-bit ASCII encoding (non-ASCII [characters] are skipped)                                              |
