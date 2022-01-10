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
| Bits iteration             | `for bit in n.bits(2..=9) { ... }`      |
| [Array] range-based API's  | `array.extract(2..8)`                   |
| [BLOB] range-based API's   | `blob.parse_le_int(4..8)`               |
| [String] range-based API's | `string.sub_string(4..=12)`             |
| [Characters] iteration     | `for ch in string.bits(4..=12) { ... }` |
| [Custom types]             | `my_obj.action(3..=15, "foo");`         |


Use as Parameter Type
---------------------

Native Rust functions that take parameters of type `std::ops::Range<INT>` or
`std::ops::RangeInclusive<INT>`, when registered into an [`Engine`], accept ranges as arguments.

However, beware that `..` (exclusive range) and `..=` (inclusive range) are _different_ types
to Rhai and they do not interoperate.  Therefore, two different versions of the same API must
be registered to handle both range styles.

```rust no_run
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

```rust no_run
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
