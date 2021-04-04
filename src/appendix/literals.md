Literals Syntax
===============

{{#include ../links.md}}

|                                    Type                                    | Literal syntax                                                                                                          |
| :------------------------------------------------------------------------: | ----------------------------------------------------------------------------------------------------------------------- |
|                                   `INT`                                    | decimal: `42`, `-123`, `0`<br/>hex: `0x????..`<br/>binary: `0b????..`<br/>octal: `0o????..`                             |
| `FLOAT`,<br/>[`Decimal`][rust_decimal] (requires [`no_float`]+[`decimal`]) | `42.0`, `-123.456`, `123.`, `123.456e-10`                                                                               |
|                              Normal [string]                               | `"... \x?? \u???? \U???????? ..."`                                                                                      |
|                         [String] with continuation                         | `"this is the first line\`<br/>`second line\`<br/>`the third line"`                                                     |
|                        Multi-line literal [string]                         | `` `this is the first line``<br/>``second line``</br>``the last line` ``                                                |
|               Multi-line literal [string] with interpolation               | `` `this is the first field: ${obj.field1}``<br/>``second field: {obj.field2}``</br>``the last field: ${obj.field3}` `` |
|                                [Character]                                 | single: `'?'`<br/>ASCII hex: `'\x??'`<br/>Unicode: `'\u????'`, `'\U????????'`                                           |
|                                 [`Array`]                                  | `[ ???, ???, ??? ]`                                                                                                     |
|                                [Object map]                                | `#{ a: ???, b: ???, c: ???, "def": ??? }`                                                                               |
|                                Boolean true                                | `true`                                                                                                                  |
|                               Boolean false                                | `false`                                                                                                                 |
|                     `Nothing`/`null`/`nil`/`void`/Unit                     | `()`                                                                                                                    |
