Strings and Characters
======================

{{#include ../links.md}}

```admonish tip.side "Safety"

Always limit the [maximum length of strings].
```

String in Rhai contain any text sequence of valid Unicode characters.

Internally strings are stored in UTF-8 encoding.

[`type_of()`] a string returns `"string"`.


String and Character Literals
-----------------------------

String and character literals follow JavaScript-style syntax.

| Type                      |   Quotes    | Escapes? | Continuation? | Interpolation? |
| ------------------------- | :---------: | :------: | :-----------: | :------------: |
| Normal string             |   `"..."`   |   yes    |   with `\`    |     **no**     |
| Multi-line literal string | `` `...` `` |  **no**  |    **no**     | with `${...}`  |
| Character                 |   `'...'`   |   yes    |    **no**     |     **no**     |

```admonish tip.small "Tip: Building strings"

Strings can be built up from other strings and types via the `+` operator
(provided by the [`MoreStringPackage`][built-in packages] but excluded when using a [raw `Engine`]).

This is particularly useful when printing output.
```


Standard Escape Sequences
-------------------------

~~~admonish tip.side "Tip: Character `to_int()`"

Use the `to_int` method to convert a Unicode character into its 32-bit Unicode encoding.
~~~

There is built-in support for Unicode (`\u`_xxxx_ or `\U`_xxxxxxxx_) and hex (`\x`_xx_) escape
sequences for normal strings and characters.

Hex sequences map to ASCII characters, while `\u` maps to 16-bit common Unicode code points and `\U`
maps the full, 32-bit extended Unicode code points.

Escape sequences are not supported for multi-line literal strings wrapped by back-ticks (`` ` ``).

| Escape sequence | Meaning                          |
| --------------- | -------------------------------- |
| `\\`            | back-slash (`\`)                 |
| `\t`            | tab                              |
| `\r`            | carriage-return (`CR`)           |
| `\n`            | line-feed (`LF`)                 |
| `\"` or `""`    | double-quote (`"`)               |
| `\'`            | single-quote (`'`)               |
| `\x`_xx_        | ASCII character in 2-digit hex   |
| `\u`_xxxx_      | Unicode character in 4-digit hex |
| `\U`_xxxxxxxx_  | Unicode character in 8-digit hex |


Line Continuation
-----------------

For a normal string wrapped by double-quotes (`"`), a back-slash (`\`) character at the end of a
line indicates that the string continues onto the next line _without any line-break_.

Whitespace up to the indentation of the opening double-quote is ignored in order to enable lining up
blocks of text.

Spaces are _not_ added, so to separate one line with the next with a space, put a space before the
ending back-slash (`\`) character.

```rust
let x = "hello, world!\
         hello world again! \
         this is the ""last"" time!!!";
// ^^^^^^ these whitespaces are ignored

// The above is the same as:
let x = "hello, world!hello world again! this is the \"last\" time!!!";
```

A string with continuation does not open up a new line.  To do so, a new-line character must be
manually inserted at the appropriate position.

```rust
let x = "hello, world!\n\
         hello world again!\n\
         this is the last time!!!";

// The above is the same as:
let x = "hello, world!\nhello world again!\nthis is the last time!!!";
```

~~~admonish warning.small "No ending quote before the line ends is a syntax error"

If the ending double-quote is omitted, it is a syntax error.

```rust
let x = "hello
# ";
//            ^ syntax error: unterminated string literal
```
~~~

```admonish question.small "Why not go multi-line?"

Technically speaking, there is no difficulty in allowing strings to run for multiple lines
_without_ the continuation back-slash.

Rhai forces you to manually mark a continuation with a back-slash because the ending quote is easy to omit.
Once it happens, the entire remainder of the script would have become one giant, multi-line string.

This behavior is different from Rust, where string literals can run for multiple lines.
```


Indexing
--------

Strings can be _indexed_ into to get access to any individual character.
This is similar to many modern languages but different from Rust.

### From beginning

Individual characters within a string can be accessed with zero-based, non-negative integer indices:

> _string_ `[` _index from 0 to (total number of characters − 1)_ `]`

### From end

A _negative_ index accesses a character in the string counting from the _end_, with −1 being the
_last_ character.

> _string_ `[` _index from −1 to −(total number of characters)_ `]`

```admonish warning.small "Character indexing can be SLOOOOOOOOW"

Internally, a Rhai string is still stored compactly as a Rust UTF-8 string in order to save memory.

Therefore, getting the character at a particular index involves walking through the entire UTF-8
encoded bytes stream to extract individual Unicode characters, counting them on the way.

Because of this, indexing can be a _slow_ procedure, especially for long strings.
Along the same lines, getting the _length_ of a string (which returns the number of characters, not
bytes) can also be slow.
```


Examples
--------

```js
let name = "Bob";
let middle_initial = 'C';
let last = "Davis";

let full_name = `${name} ${middle_initial}. ${last}`;
full_name == "Bob C. Davis";

// String building with different types
let age = 42;
let record = `${full_name}: age ${age}`;
record == "Bob C. Davis: age 42";

// Unlike Rust, Rhai strings can be indexed to get a character
// (disabled with 'no_index')
let c = record[4];
c == 'C';

ts.s = record;                          // custom type properties can take strings

let c = ts.s[4];
c == 'C';

let c = ts.s[-4];                       // negative index counts from the end
c == 'e';

let c = "foo"[0];                       // indexing also works on string literals...
c == 'f';

let c = ("foo" + "bar")[5];             // ... and expressions returning strings
c == 'r';

// Escape sequences in strings
record += " \u2764\n";                  // escape sequence of '❤' in Unicode
record == "Bob C. Davis: age 42 ❤\n";  // '\n' = new-line

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
