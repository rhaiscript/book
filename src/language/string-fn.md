Built-in String Functions
========================

{{#include ../links.md}}


Standard Functions
------------------

The following standard methods (mostly defined in the [`MoreStringPackage`][packages] but excluded if
using a [raw `Engine`]) operate on [strings]:

| Function                  | Not available under | Parameter(s)                                                                           | Description                                                                                                          |
| ------------------------- | :-----------------: | -------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------- |
| `len` method and property |                     | _none_                                                                                 | returns the number of characters (not number of bytes) in the string                                                 |
| `pad`                     |                     | 1) target length<br/>2) character/string to pad                                        | pads the string with a character or a string to at least a specified length                                          |
| `append`                  |                     | character/string to append                                                             | adds a character or a string to the end of another string                                                            |
| `remove`                  |                     | character/string to remove                                                             | removes a character or a string from the string                                                                      |
| `clear`                   |                     | _none_                                                                                 | empties the string                                                                                                   |
| `truncate`                |                     | target length                                                                          | cuts off the string at exactly a specified number of characters                                                      |
| `contains`                |                     | character/sub-string to search for                                                     | checks if a certain character or sub-string occurs in the string                                                     |
| `index_of`                |                     | 1) character/sub-string to search for<br/>2) _(optional)_ start index                  | returns the index that a certain character or sub-string occurs in the string, or -1 if not found                    |
| `sub_string`              |                     | 1) start index<br/>2) _(optional)_ number of characters to extract, none if < 0        | extracts a sub-string (to the end of the string if length is not specified)                                          |
| `split`                   |    [`no_index`]     | _none_                                                                                 | splits the string by individual characters, returning an [array] of characters                                       |
| `split`                   |    [`no_index`]     | Position to split at (in number of characters), beginning if < 0, end if ≥ length      | splits the string into two segments at the specified character position, returning an [array] of two string segments |
| `split`                   |    [`no_index`]     | 1) delimiter character/string<br/>2) _(optional)_ maximum number of segments, 1 if < 1 | splits the string by the specified delimiter, returning an [array] of string segments                                |
| `split_rev`               |    [`no_index`]     | 1) delimiter character/string<br/>2) _(optional)_ maximum number of segments, 1 if < 1 | splits the string by the specified delimiter in reverse order, returning an [array] of string segments               |
| `crop`                    |                     | 1) start index<br/>2) _(optional)_ number of characters to retain, none if < 0         | retains only a portion of the string                                                                                 |
| `replace`                 |                     | 1) target character/sub-string<br/>2) replacement character/string                     | replaces a sub-string with another                                                                                   |
| `trim`                    |                     | _none_                                                                                 | trims the string of whitespace at the beginning and end                                                              |


Standard Operators
------------------

The following standard operators inter-operate between [strings] and/or [characters][strings].

When one (or both) of the operands is a [character], it is first converted into a one-character
[string] before running the operator.

| Operator  | Description                             |
| --------- | --------------------------------------- |
| `+`, `+=` | character/string concatenation          |
| `-`, `-=` | remove character/sub-string from string |
| `==`      | equals to                               |
| `!=`      | not equals to                           |
| `>`       | greater than                            |
| `>=`      | greater than or equals to               |
| `<`       | less than                               |
| `<=`      | less than or equals to                  |


Examples
--------

```rust,no_run
let full_name == " Bob C. Davis ";
full_name.len == 14;

full_name.trim();
full_name.len == 12;
full_name == "Bob C. Davis";

full_name.pad(15, '$');
full_name.len == 15;
full_name == "Bob C. Davis$$$";

let n = full_name.index_of('$');
n == 12;

full_name.index_of("$$", n + 1) == 13;

full_name.sub_string(n, 3) == "$$$";

full_name.truncate(6);
full_name.len == 6;
full_name == "Bob C.";

full_name.replace("Bob", "John");
full_name.len == 7;
full_name == "John C.";

full_name.contains('C') == true;
full_name.contains("John") == true;

full_name.crop(5);
full_name == "C.";

full_name.crop(0, 1);
full_name == "C";

full_name.clear();
full_name.len == 0;
```
