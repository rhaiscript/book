The `ImmutableString` Type
==========================

{{#include ../links.md}}

All [strings] in Rhai are implemented as `ImmutableString`, which is an alias to
`Rc<SmartString>` (or `Arc<SmartString>` under [`sync`]).

[`SmartString`] is used because many strings in scripts are short (fewer than 24 ASCII characters).

An `ImmutableString` is immutable (i.e. never changes) and therefore can be shared among many users.
Cloning an `ImmutableString` is cheap since it only copies an immutable reference.

Modifying an `ImmutableString` causes it first to be cloned, and then the modification made to the copy.
Therefore, direct string modifications are expensive.


Avoid `String` Parameters
-------------------------

`ImmutableString` should be used in place of `String` for function parameters because using `String`
is very inefficient (the argument is cloned during every function call).

A alternative is to use `&str` which de-sugars to `ImmutableString`.

A function with the first parameter being `&mut String` does not match a string argument passed to it,
which has type `ImmutableString`.  In fact, `&mut String` is treated as an opaque [custom type].

```rust no_run
fn slow(s: String) -> i64 { ... }               // string is cloned each call

fn fast1(s: ImmutableString) -> i64 { ... }     // cloning 'ImmutableString' is cheap

fn fast2(s: &str) -> i64 { ... }                // de-sugars to above

fn bad(s: &mut String) { ... }                  // '&mut String' will not match string values
```


Differences from Rust Strings
----------------------------

Internally Rhai strings are stored as UTF-8 just like Rust (they _are_ Rust `String`s!),
but nevertheless there are major differences.

In Rhai a [string] is semantically the same as an array of Unicode characters and can be directly
indexed (unlike Rust).

This is similar to most other languages (e.g. JavaScript, C#) where strings are stored internally
not as UTF-8 but as arrays of UCS-16 or UCS-32.

Individual characters within a Rhai string can also be replaced just as if the string is an array of
Unicode characters.

In Rhai, there are also no separate concepts of `String` and `&str` (a string slice) as in Rust.


Performance Considerations of Character Indexing
-----------------------------------------------

Although Rhai exposes a [string] as a simple array of `char` which can be directly indexed to get at
a particular character, internally the [string] is still stored as UTF-8 (native Rust `String`s).

All indexing operations require walking through the entire UTF-8 string to find the offset of the
particular character position, and therefore is _much_ slower than the simple array indexing for
other scripting languages.

This implementation detail is hidden from the user but has a performance implication.

Avoid large scale character-based processing of [strings]; instead, build an actual [array] of
[characters] (via the `split()` method) which can then be manipulated efficiently.
