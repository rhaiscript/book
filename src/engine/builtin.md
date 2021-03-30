Built-in Operators
==================

{{#include ../links.md}}

The following operators are built-in, meaning that they are always available,
even when using a [raw `Engine`].

| Operators                 | Assignment operators          | Supported types<br/>(see [standard types])                                                                                                                                                           |
| ------------------------- | ----------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `+`,                      | `+=`                          | &bull; `INT`<br/>&bull; `FLOAT` (if not [`no_float`])<br/>&bull; [`Decimal`][rust_decimal] (requires [`decimal`])<br/>&bull; `char`<br/>&bull; [`ImmutableString`]                                   |
| `-`, `*`, `/`, `%`, `**`, | `-=`, `*=`, `/=`, `%=`, `**=` | &bull; `INT`<br/>&bull; `FLOAT` (if not [`no_float`])<br/>&bull; [`Decimal`][rust_decimal] (requires [`decimal`])                                                                                    |
| `<<`, `>>`                | `<<=`, `>>=`                  | &bull; `INT`                                                                                                                                                                                         |
| `&`, <code>\|</code>, `^` | `&=`, <code>\|=</code>, `^=`  | &bull; `INT` (bit-wise)<br/>&bull; `bool` (non-short-circuiting)                                                                                                                                     |
| `&&`, <code>\|\|</code>   |                               | &bull; `bool` (short-circuits)                                                                                                                                                                       |
| `==`, `!=`                |                               | &bull; `INT`<br/>&bull; `FLOAT` (if not [`no_float`])<br/>&bull; [`Decimal`][rust_decimal] (requires [`decimal`])<br/>&bull; `bool`<br/>&bull; `char`<br/>&bull; [`ImmutableString`]<br/>&bull; `()` |
| `>`, `>=`, `<`, `<=`      |                               | &bull; `INT`<br/>&bull; `FLOAT` (if not [`no_float`])<br/>&bull; [`Decimal`][rust_decimal] (requires [`decimal`])<br/>&bull; `char`<br/>&bull; [`ImmutableString`]<br/>&bull; `()`                   |
| `in`                      |                               | &bull; [`ImmutableString`]<br/>&bull; `char`/[`ImmutableString`]<br/>&bull; [`ImmutableString`]/[object map]                                                                                         |

All built-in operators are binary, and are supported for both operands of the same type.

`FLOAT` and [`Decimal`][rust_decimal] also inter-operate with `INT`, while [strings] inter-operate
with [characters][string] for certain operators (e.g. `+`).
