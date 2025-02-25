Ranges
======

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
| Bits iteration             | `for bit in n.bits(2..=9) { ... }`      |
| [Array] range-based APIs   | `array.extract(2..8)`                   |
| [BLOB] range-based APIs    | `blob.parse_le_int(4..8)`               |
| [String] range-based APIs  | `string.sub_string(4..=12)`             |
| [Characters] iteration     | `for ch in string.bits(4..=12) { ... }` |
| [Custom types]             | `my_obj.action(3..=15, "foo");`         |


Use as Parameter Type
---------------------

Native Rust functions that take parameters of type `std::ops::Range<INT>` or
`std::ops::RangeInclusive<INT>`, when registered into an [`Engine`], accept ranges as arguments.

```admonish warning.small "Different types"

`..` (exclusive range) and `..=` (inclusive range) are _different_ types to Rhai
and they do not interoperate.

Two different versions of the same API must be registered to handle both range styles.
```

```rust
use std::ops::{Range, RangeInclusive};

/// The actual work function
fn do_work(obj: &mut TestStruct, from: i64, to: i64, inclusive: bool) {
    ...
}

let mut engine = Engine::new();

engine
    /// Version of API that accepts an exclusive range
    .register_fn("do_work", |obj: &mut TestStruct, range: Range<i64>|
        do_work(obj, range.start, range.end, false)
    )
    /// Version of API that accepts an inclusive range
    .register_fn("do_work", |obj: &mut TestStruct, range: RangeInclusive<i64>|
        do_work(obj, range.start(), range.end(), true)
    );

engine.run(
"
    let obj = new_ts();

    obj.do_work(0..12);         // use exclusive range

    obj.do_work(0..=11);        // use inclusive range
")?;
```

### Indexers Using Ranges

[Indexers] commonly use ranges as parameters.

```rust
use std::ops::{Range, RangeInclusive};

let mut engine = Engine::new();

engine
    /// Version of indexer that accepts an exclusive range
    .register_indexer_get_set(
        |obj: &mut TestStruct, range: Range<i64>| -> bool { ... },
        |obj: &mut TestStruct, range: Range<i64>, value: bool| { ... },
    )
    /// Version of indexer that accepts an inclusive range
    .register_indexer_get_set(
        |obj: &mut TestStruct, range: RangeInclusive<i64>| -> bool { ... },
        |obj: &mut TestStruct, range: RangeInclusive<i64>, value: bool| { ... },
    );

engine.run(
"
    let obj = new_ts();

    let x = obj[0..12];         // use exclusive range

    obj[0..=11] = !x;           // use inclusive range
")?;
```

Built-in Functions
------------------

The following methods (mostly defined in the [`BasicIteratorPackage`][built-in packages] but
excluded when using a [raw `Engine`]) operate on ranges.

| Function                           |  Parameter(s)   | Description                                   |
| ---------------------------------- | :-------------: | --------------------------------------------- |
| `start` method and property        |                 | beginning of the range                        |
| `end` method and property          |                 | end of the range                              |
| `contains`, [`in`] operator        | number to check | does this range contain the specified number? |
| `is_empty` method and property     |                 | returns `true` if the range contains no items |
| `is_inclusive` method and property |                 | is the range inclusive?                       |
| `is_exclusive` method and property |                 | is the range exclusive?                       |


TL;DR
-----

```admonish question "What happened to the _open-ended_ ranges?"

Rust has _open-ended_ ranges, such as `start..`, `..end` and `..=end`.  They are not available in Rhai.

They are not needed because Rhai can [overload][function overloading] functions.

Typically, an API accepting ranges as parameters would have equivalent versions that accept a
starting position and a length (the standard `start + len` pair), as well as a versions that accept
only the starting position (the length assuming to the end).

In fact, usually all versions redirect to a call to one single version.

For example, a naive implementation of the `extract` method for [arrays] (without any error handling)
would look like:

~~~rust
use std::ops::{Range, RangeInclusive};

// Version with exclusive range
#[rhai_fn(name = "extract", pure)]
pub fn extract_range(array: &mut Array, range: Range<i64>) -> Array {
    array[range].to_vec()
}
// Version with inclusive range
#[rhai_fn(name = "extract", pure)]
pub fn extract_range2(array: &mut Array, range: RangeInclusive<i64>) -> Array {
    extract_range(array, range.start()..range.end() + 1)
}
// Version with start
#[rhai_fn(name = "extract", pure)]
pub fn extract_to_end(array: &mut Array, start: i64) -> Array {
    extract_range(array, start..start + array.len())
}
// Version with start+len
#[rhai_fn(name = "extract", pure)]
pub fn extract(array: &mut Array, start: i64, len: i64) -> Array {
    extract_range(array, start..start + len)
}
~~~

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
