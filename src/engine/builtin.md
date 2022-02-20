Built-in Operators
==================

{{#include ../links.md}}

The following operators are built-in, meaning that they are always available, even when using a [raw `Engine`].

All built-in operators are binary, and are supported for both operands of the same type.

| Operators                 | Assignment operators          | Supported types<br/>(see [standard types])                                                                                                                                                                                             |
| ------------------------- | ----------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `+`,                      | `+=`                          | &bull; `INT`<br/>&bull; `FLOAT` (if not [`no_float`])<br/>&bull; [`Decimal`][rust_decimal] (requires [`decimal`])<br/>&bull; `char`<br/>&bull; [string]                                                                                |
| `-`, `*`, `/`, `%`, `**`, | `-=`, `*=`, `/=`, `%=`, `**=` | &bull; `INT`<br/>&bull; `FLOAT` (if not [`no_float`])<br/>&bull; [`Decimal`][rust_decimal] (requires [`decimal`])                                                                                                                      |
| `<<`, `>>`                | `<<=`, `>>=`                  | &bull; `INT`                                                                                                                                                                                                                           |
| `&`, <code>\|</code>, `^` | `&=`, <code>\|=</code>, `^=`  | &bull; `INT` (bit-wise)<br/>&bull; `bool` (non-short-circuiting)                                                                                                                                                                       |
| `&&`, <code>\|\|</code>   |                               | &bull; `bool` (short-circuits)                                                                                                                                                                                                         |
| `==`, `!=`                |                               | &bull; `INT`<br/>&bull; `FLOAT` (if not [`no_float`])<br/>&bull; [`Decimal`][rust_decimal] (requires [`decimal`])<br/>&bull; `bool`<br/>&bull; `char`<br/>&bull; [string]<br/>&bull; [BLOB]<br/>&bull; numeric [range]<br/>&bull; `()` |
| `>`, `>=`, `<`, `<=`      |                               | &bull; `INT`<br/>&bull; `FLOAT` (if not [`no_float`])<br/>&bull; [`Decimal`][rust_decimal] (requires [`decimal`])<br/>&bull; `char`<br/>&bull; [string]<br/>&bull; `()`                                                                |
| [`in`]                    |                               | &bull; [string]<br/>&bull; `char`/[string]<br/>&bull; [string]/[object map] (if not [`no_object`])<br/>&bull;`INT`/[BLOB] (if not [`no_index`])<br/>&bull;`INT`/numeric [range]                                                        |

```admonish note

`FLOAT` and [`Decimal`][rust_decimal] also inter-operate with `INT`, while [strings] inter-operate
with [characters][string] for certain operators (e.g. `+`).
```
