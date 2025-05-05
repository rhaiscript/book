Operator Precedence
===================

{{#include ../links.md}}

All operators in Rhai has a _precedence_ indicating how tightly they bind.

A higher precedence binds more tightly than a lower precedence, so `*` and `/` binds before `+` and `-` etc.

When registering a custom operator, the operator's precedence must also be provided.

The following _precedence table_ shows the built-in precedence of standard Rhai operators:

| Category            |      Operators       | Binding | Precedence (0-255) |
| ------------------- | :------------------: | :-----: | :----------------: |
| Logic and bit masks |  `\|\|`,  `\|`, `^`  |  left   |         30         |
| Logic and bit masks |      `&&`, `&`       |  left   |         60         |
| Comparisons         |      `==`, `!=`      |  left   |         90         |
| Containment         |        [`in`]        |  left   |        110         |
| Comparisons         | `>`, `>=`, `<`, `<=` |  left   |        130         |
| Null-coalesce       |         `??`         |  left   |        135         |
| Ranges              |     `..`, `..=`      |  left   |        140         |
| Arithmetic          |       `+`, `-`       |  left   |        150         |
| Arithmetic          |    `*`, `/`, `%`     |  left   |        180         |
| Arithmetic          |         `**`         |  right  |        190         |
| Bit-shifts          |      `<<`, `>>`      |  left   |        210         |
| Unary operators     |    `+`, `-`, `!`     |  right  |      highest       |
| Object field access |      `.`, `?.`       |  right  |      highest       |
