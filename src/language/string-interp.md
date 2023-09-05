Multi-Line Literal Strings
==========================

{{#include ../links.md}}

A [string] wrapped by a pair of back-tick (`` ` ``) characters is interpreted _literally_.

This means that every single character that lies between the two back-ticks is taken verbatim.

This include new-lines, whitespaces, escape characters etc.

```js
let x = `hello, world! "\t\x42"
  hello world again! 'x'
     this is the last time!!! `;

// The above is the same as:
let x = "hello, world! \"\\t\\x42\"\n  hello world again! 'x'\n     this is the last time!!! ";
```

If a back-tick (`` ` ``) appears at the _end_ of a line, then it is understood that the entire text
block starts from the _next_ line; the starting new-line character is stripped.

```js
let x = `
        hello, world! "\t\x42"
  hello world again! 'x'
     this is the last time!!!
`;

// The above is the same as:
let x = "        hello, world! \"\\t\\x42\"\n  hello world again! 'x'\n     this is the last time!!!\n";
```

To actually put a back-tick (`` ` ``) character inside a multi-line literal [string], use two
back-ticks together (i.e. ` `` `).

```js
let x = `I have a quote " as well as a back-tick `` here.`;

// The above is the same as:
let x = "I have a quote \" as well as a back-tick ` here.";
```


String Interpolation
====================

```admonish warning.side "Only literal strings"

Interpolation is not supported for normal [string] or [character] literals.
```

Multi-line literal [strings] support _string interpolation_ wrapped in `${` ... `}`.

`${` ... `}` acts as a statements _block_ and can contain anything that is allowed within a
statements block, including another interpolated [string]!
The last result of the block is taken as the value for interpolation.

Rhai uses [`to_string`] to convert any value into a [string], then physically joins all the
sub-strings together.

For convenience, if any interpolated value is a [BLOB], however, it is automatically treated as a
UTF-8 encoded string.  That is because it is rarely useful to interpolate a [BLOB] into a [string],
but extremely useful to be able to directly manipulate UTF-8 encoded text.

```js
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

let blob = blob(3, 0x21);

print(blob);                            // prints [212121]

print(`Data: ${blob}`);                 // prints "Data: !!!"
                                        // BLOB is treated as UTF-8 encoded string

print(`Data: ${blob.to_string()}`);     // prints "Data: [212121]"
```

~~~admonish question.small "What if I want `${` inside?"

🤦 Well, you just _have_ to ask for the impossible, don't you?

Currently there is no way to escape `${`.  Build the [string] in three pieces:

```js
`Interpolations start with "` + "${" + `" and end with }.`
```
~~~

