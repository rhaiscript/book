Built-in Operators
==================

{{#include ../links.md}}

The following operators are built-in, meaning that they are always available, even when using a [raw `Engine`].

All built-in operators are binary, and are supported for both operands of the same type.

| Operators                 | Assignment operators          | Supported types<br/>(see [standard types])                                                                                                                                                                                |
| ------------------------- | ----------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `+`,                      | `+=`                          | <ul><li>`INT`</li><li>`FLOAT` (if not [`no_float`])</li><li>[`Decimal`][rust_decimal] (requires [`decimal`])</li><li>`char`</li><li>[string]</li></ul>                                                                    |
| `-`, `*`, `/`, `%`, `**`, | `-=`, `*=`, `/=`, `%=`, `**=` | <ul><li>`INT`</li><li>`FLOAT` (if not [`no_float`])</li><li>[`Decimal`][rust_decimal] (requires [`decimal`])</li></ul>                                                                                                    |
| `<<`, `>>`                | `<<=`, `>>=`                  | <ul><li>`INT`</li></ul>                                                                                                                                                                                                   |
| `&`, `\|`, `^`            | `&=`, `\|=`, `^=`             | <ul><li>`INT` (bit-wise)</li><li>`bool` (non-short-circuiting)</li></ul>                                                                                                                                                  |
| `&&`, `\|\|`              |                               | <ul><li>`bool` (short-circuits)</li></ul>                                                                                                                                                                                 |
| `==`, `!=`                |                               | <ul><li>`INT`</li><li>`FLOAT` (if not [`no_float`])</li><li>[`Decimal`][rust_decimal] (requires [`decimal`])</li><li>`bool`</li><li>`char`</li><li>[string]</li><li>[BLOB]</li><li>numeric [range]</li><li>`()`</li></ul> |
| `>`, `>=`, `<`, `<=`      |                               | <ul><li>`INT`</li><li>`FLOAT` (if not [`no_float`])</li><li>[`Decimal`][rust_decimal] (requires [`decimal`])</li><li>`char`</li><li>[string]</li><li>`()`</li></ul>                                                       |

```admonish tip.small

`FLOAT` and [`Decimal`][rust_decimal] also inter-operate with `INT`, while [strings] inter-operate
with [characters][string] for certain operators (e.g. `+`).
```
