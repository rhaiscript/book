Literals Syntax
===============

{{#include ../links.md}}

|                Type                |                                       Literal syntax                                        |
| :--------------------------------: | :-----------------------------------------------------------------------------------------: |
|               `INT`                | decimal: `42`, `-123`, `0`<br/>hex: `0x????..`<br/>binary: `0b????..`<br/>octal: `0o????..` |
|              `FLOAT`               |                          `42.0`, `-123.456`, `0.0`, `123.456e-10`                           |
|              [String]              |                             `"... \x?? \u???? \U???????? ..."`                              |
|        [Character][string]         |        single: `'?'`<br/>ASCII hex: `'\x??'`<br/>Unicode: `'\u????'`, `'\U????????'`        |
|             [`Array`]              |                                     `[ ???, ???, ??? ]`                                     |
|            [Object map]            |                          `#{ a: ???, b: ???, c: ???, "def": ??? }`                          |
|            Boolean true            |                                           `true`                                            |
|           Boolean false            |                                           `false`                                           |
| `Nothing`/`null`/`nil`/`void`/Unit |                                            `()`                                             |
