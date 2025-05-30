Operators and Symbols
=====================

{{#include ../links.md}}


Operators
---------

|                                     Operator                                      | Description                            |  Binary?   | Binding direction |
| :-------------------------------------------------------------------------------: | -------------------------------------- | :--------: | :---------------: |
|                                        `+`                                        | add                                    |    yes     |       left        |
|                                        `-`                                        | 1) subtract<br/>2) negative (prefix)   | yes<br/>no |  left<br/>right   |
|                                        `*`                                        | multiply                               |    yes     |       left        |
|                                        `/`                                        | divide                                 |    yes     |       left        |
|                                        `%`                                        | modulo                                 |    yes     |       left        |
|                                       `**`                                        | power/exponentiation                   |    yes     |       right       |
|                                       `>>`                                        | right bit-shift                        |    yes     |       left        |
|                                       `<<`                                        | left bit-shift                         |    yes     |       left        |
|                                        `&`                                        | 1) bit-wise _AND_<br/>2) boolean _AND_ |    yes     |       left        |
|                                       `\|`                                        | 1) bit-wise _OR_<br/>2) boolean _OR_   |    yes     |       left        |
|                                        `^`                                        | 1) bit-wise _XOR_<br/>2) boolean _XOR_ |    yes     |       left        |
| `=`, `+=`, `-=`, `*=`, `/=`,<br/>`**=`, `%=`, `<<=`, `>>=`, `&=`,<br/>`\|=`, `^=` | assignments                            |    yes     |        n/a        |
|                                       `==`                                        | equals to                              |    yes     |       left        |
|                                       `!=`                                        | not equals to                          |    yes     |       left        |
|                                        `>`                                        | greater than                           |    yes     |       left        |
|                                       `>=`                                        | greater than or equals to              |    yes     |       left        |
|                                        `<`                                        | less than                              |    yes     |       left        |
|                                       `<=`                                        | less than or equals to                 |    yes     |       left        |
|                                       `&&`                                        | boolean _AND_ (short-circuits)         |    yes     |       left        |
|                                      `\|\|`                                       | boolean _OR_ (short-circuits)          |    yes     |       left        |
|                                       `??`                                        | null-coalesce (short-circuits)         |    yes     |       left        |
|                                        `!`                                        | boolean _NOT_                          |   **no**   |       right       |
|                             `[` ... `]`, `?[` ... `]`                             | indexing                               |    yes     |       left        |
|                                     `.`, `?.`                                     | 1) property access<br/>2) method call  |    yes     |       left        |
|                                       `..`                                        | exclusive range                        |    yes     |       left        |
|                                       `..=`                                       | inclusive range                        |    yes     |       left        |


Symbols and Patterns
--------------------

|       Symbol        | Name                               | Description                           |
| :-----------------: | ---------------------------------- | ------------------------------------- |
|         `_`         | underscore                         | default `switch` case                 |
|         `;`         | semicolon                          | statement separator                   |
|         `,`         | comma                              | list separator                        |
|         `:`         | colon                              | [object map] property value separator |
|        `::`         | path                               | module path separator                 |
|    `#{` ... `}`     | hash map                           | [object map] literal                  |
|     `"` ... `"`     | double quote                       | [string]                              |
| `` ` `` ... `` ` `` | back-tick                          | multi-line literal [string]           |
|     `'` ... `'`     | single quote                       | [character]                           |
|         `\`         | 1) escape<br/>2) line continuation | escape character literal              |
|        `()`         | unit                               | null value                            |
|     `(` ... `)`     | parentheses                        | expression grouping                   |
|     `{` ... `}`     | braces                             | block statement                       |
|    `\|` ... `\|`    | pipes                              | closure                               |
|     `[` ... `]`     | brackets                           | [array] literal                       |
|         `!`         | bang                               | function call in calling scope        |
|        `=>`         | double arrow                       | `switch` expression case separator    |
|        `//`         | comment                            | line comment                          |
|        `///`        | doc-comment                        | line [doc-comment]                    |
|        `//!`        | module doc                         | [module documentation][comments]      |
|    `/*` ... `*/`    | comment                            | block comment                         |
|   `/**` ... `*/`    | doc-comment                        | block [doc-comment]                   |
|    `(*` ... `*)`    | comment                            | _reserved_                            |
|        `#!`         | shebang                            | _reserved_                            |
|        `++`         | increment                          | _reserved_                            |
|        `--`         | decrement                          | _reserved_                            |
|        `...`        | rest                               | _reserved_                            |
|         `~`         | tilde                              | _reserved_                            |
|        `!.`         |                                    | _reserved_                            |
|         `?`         | question                           | _reserved_                            |
|         `#`         | hash                               | _reserved_                            |
|         `@`         | at                                 | _reserved_                            |
|         `$`         | dollar                             | _reserved_                            |
|        `->`         | arrow                              | _reserved_                            |
|        `<-`         | left arrow                         | _reserved_                            |
|        `<\|`        | left triangle                      | _reserved_                            |
|        `\|>`        | right triangle                     | _reserved_                            |
|        `===`        | strict equals to                   | _reserved_                            |
|        `!==`        | strict not equals to               | _reserved_                            |
|        `:=`         | assignment                         | _reserved_                            |
|        `:;`         | typo to `::`                       | _reserved_                            |
|    `::<` ... `>`    | turbofish                          | _reserved_                            |
