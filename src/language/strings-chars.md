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


String and Character Literals
----------------------------

String and character literals follow JavaScript-style formatting:

* normal strings are wrapped by double-quotes: `"`

* multi-line literal strings are wrapped by back-ticks: `` ` ``

* characters are wrapped by single-quotes: `'`


Standard Escape Sequences
-------------------------

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
| `\"`            | double-quote (`"`)               |
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

```rust,no_run
let x = "hello, world!\
         hello world again! \
         this is the last time!!!";

// ^^^^^^ these whitespaces are ignored

// The above is the same as:
let x = "hello, world!hello world again! this is the last time!!!";
```

A string with continuation does not open up a new line.  To do so, a new-line character must be
manually inserted at the appropriate position:

```rust,no_run
let x = "hello, world!\n\
         hello world again!\n\
         this is the last time!!!";

// The above is the same as:
let x = "hello, world!\nhello world again!\nthis is the last time!!!";
```


Multi-Line Literal Strings
--------------------------

A string wrapped by a pair of back-tick (`` ` ``) characters is interpreted _literally_,
meaning that every single character that lies between the two back-ticks is taken verbatim.
This include new-lines, whitespaces, escape characters etc.

```js,no_run
let x = `hello, world! "\t\x42"
  hello world again! 'x'
     this is the last time!!! `;

// The above is the same as:
let x = "hello, world! \"\\t\\x42\"\n  hello world again! 'x'\n     this is the last time!!! ";
```

If a back-tick (`` ` ``) appears at the _end_ of a line, then it is understood that the entire text
block starts from the _next_ line; the starting new-line character is stripped.

```js,no_run
let x = `
        hello, world! "\t\x42"
  hello world again! 'x'
     this is the last time!!!
`;

// The above is the same as:
let x = "        hello, world! \"\\t\\x42\"\n  hello world again! 'x'\n     this is the last time!!!\n";
```

To actually put a back-tick (`` ` ``) character inside a multi-line literal string requires post-processing.


String Interpolation
--------------------

Multi-line literal strings support _string interpolation_ wrapped in `${` ... `}`.

Interpolation is not supported for normal string or character literals.

`${` ... `}` acts as a statements _block_ and can contain anything that is allowed within a
statements block, including another interpolated string!
The last result of the block is taken as the value for interpolation.

Rhai uses [`to_string()`] to convert any value into a string, then physically joins all the
sub-strings together.

```js,no_run
let x = 42;
let y = 123;

let s = `x = ${x} and y = ${y}.`;                   // <- interpolated string

let s = ("x = " + {x} + " and y = " + {y} + ".");   // <- de-sugars to this

s == "x = 42 and y = 123.";

let s = `
Undeniable logic:
1) Hello, ${let w = `${x} world`; if x > 1 { w += "s" } w}!
2) If ${y} > ${x} then it is ${y > x}!
`;

s == "Undeniable logic:\n1) Hello, 42 worlds!\n2) If 123 > 42 then it is true!\n";
```


Examples
--------

```js,no_run
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
