Raw `Engine`
===========

{{#include ../links.md}}

`Engine::new` creates a scripting [`Engine`] with common functionalities (e.g. printing to the console via `print`).

In many controlled embedded environments, however, these may not be needed and unnecessarily occupy
application code storage space.

Use `Engine::new_raw` to create a _raw_ `Engine`, in which only a minimal set of
basic arithmetic and logical operators are supported (see below).

To add more functionalities to a _raw_ `Engine`, load [packages] into it.


Built-in Operators
------------------

The following operators are built-in, meaning that they are always available.

| Operators                 | Assignment operators          | Supported types<br/>(see [standard types])                                                                                        |
| ------------------------- | ----------------------------- | --------------------------------------------------------------------------------------------------------------------------------- |
| `+`,                      | `+=`                          | `INT`, `FLOAT` (if not [`no_float`]), [`Decimal`][rust_decimal] (requires [`decimal`]), `char`, [`ImmutableString`]               |
| `-`, `*`, `/`, `%`, `**`, | `-=`, `*=`, `/=`, `%=`, `**=` | `INT`, `FLOAT` (if not [`no_float`]), [`Decimal`][rust_decimal] (requires [`decimal`])                                            |
| `<<`, `>>`                | `<<=`, `>>=`                  | `INT`                                                                                                                             |
| `&`, <code>\|</code>, `^` | `&=`, <code>\|=</code>, `^=`  | `INT`, `bool`                                                                                                                     |
| `&&`, <code>\|\|</code>   |                               | `bool`                                                                                                                            |
| `==`, `!=`                |                               | `INT`, `FLOAT` (if not [`no_float`]), [`Decimal`][rust_decimal] (requires [`decimal`]), `bool`, `char`, `()`, [`ImmutableString`] |
| `>`, `>=`, `<`, `<=`      |                               | `INT`, `FLOAT` (if not [`no_float`]), [`Decimal`][rust_decimal] (requires [`decimal`]), `char`, `()`, [`ImmutableString`]         |

All built-in operators are binary, and are supported for both operands of the same type.

`FLOAT` and [`Decimal`][rust_decimal] also inter-operate with `INT`, while [strings] inter-operate
with [characters][string] for certain operators (e.g. `+`).
