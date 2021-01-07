Strings and Characters
=====================

{{#include ../links.md}}

String in Rhai contain any text sequence of valid Unicode characters.
Internally strings are stored in UTF-8 encoding.

Strings can be built up from other strings and types via the `+` operator
(provided by the [`MoreStringPackage`][packages] but excluded if using a [raw `Engine`]).
This is particularly useful when printing output.

[`type_of()`] a string returns `"string"`.

The maximum allowed length of a string can be controlled via `Engine::set_max_string_size`
(see [maximum length of strings]).


The `ImmutableString` Type
-------------------------

All strings in Rhai are implemented as `ImmutableString` (see [standard types]).
An `ImmutableString` does not change and can be shared.

Modifying an `ImmutableString` causes it first to be cloned, and then the modification made to the copy.

### **IMPORTANT** &ndash; Avoid `String` Parameters

`ImmutableString` should be used in place of `String` for function parameters because using
`String` is very inefficient (the `String` argument is cloned during every call).

A alternative is to use `&str` which de-sugars to `ImmutableString`.

```rust
fn slow(s: String) -> i64 { ... }               // string is cloned each call

fn fast1(s: ImmutableString) -> i64 { ... }     // cloning 'ImmutableString' is cheap

fn fast2(s: &str) -> i64 { ... }                // de-sugars to above
```


String and Character Literals
----------------------------

String and character literals follow C-style formatting, with support for Unicode ('`\u`_xxxx_' or '`\U`_xxxxxxxx_')
and hex ('`\x`_xx_') escape sequences.

Hex sequences map to ASCII characters, while '`\u`' maps to 16-bit common Unicode code points and '`\U`' maps the full,
32-bit extended Unicode code points.

Standard escape sequences:

| Escape sequence | Meaning                          |
| --------------- | -------------------------------- |
| `\\`            | back-slash `\`                   |
| `\t`            | tab                              |
| `\r`            | carriage-return `CR`             |
| `\n`            | line-feed `LF`                   |
| `\"`            | double-quote `"`                 |
| `\'`            | single-quote `'`                 |
| `\x`_xx_        | ASCII character in 2-digit hex   |
| `\u`_xxxx_      | Unicode character in 4-digit hex |
| `\U`_xxxxxxxx_  | Unicode character in 8-digit hex |


Differences from Rust Strings
----------------------------

Internally Rhai strings are stored as UTF-8 just like Rust (they _are_ Rust `String`'s!),
but nevertheless there are major differences.

In Rhai a string is the same as an array of Unicode characters and can be directly indexed (unlike Rust).

This is similar to most other languages where strings are internally represented not as UTF-8 but as arrays of multi-byte
Unicode characters.

Individual characters within a Rhai string can also be replaced just as if the string is an array of Unicode characters.

In Rhai, there are also no separate concepts of `String` and `&str` as in Rust.


Examples
--------

```rust
let name = "Bob";
let middle_initial = 'C';
let last = "Davis";

let full_name = name + " " + middle_initial + ". " + last;
full_name == "Bob C. Davis";

// String building with different types
let age = 42;
let record = full_name + ": age " + age;
record == "Bob C. Davis: age 42";

// Unlike Rust, Rhai strings can be indexed to get a character
// (disabled with 'no_index')
let c = record[4];
c == 'C';

ts.s = record;                          // custom type properties can take strings

let c = ts.s[4];
c == 'C';

let c = "foo"[0];                       // indexing also works on string literals...
c == 'f';

let c = ("foo" + "bar")[5];             // ... and expressions returning strings
c == 'r';

// Escape sequences in strings
record += " \u2764\n";                  // escape sequence of '❤' in Unicode
record == "Bob C. Davis: age 42 ❤\n";   // '\n' = new-line

// Unlike Rust, Rhai strings can be directly modified character-by-character
// (disabled with 'no_index')
record[4] = '\x58'; // 0x58 = 'X'
record == "Bob X. Davis: age 42 ❤\n";

// Use 'in' to test if a substring (or character) exists in a string
"Davis" in record == true;
'X' in record == true;
'C' in record == false;

// Strings can be iterated with a 'for' statement, yielding characters
for ch in record {
    print(ch);
}
```
