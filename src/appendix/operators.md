Operators and Symbols
====================

{{#include ../links.md}}


Operators
---------

|                                          Operator                                           | Description                            |  Binary?   | Binding direction |
| :-----------------------------------------------------------------------------------------: | -------------------------------------- | :--------: | :---------------: |
|                                             `+`                                             | add                                    |    yes     |       left        |
|                                             `-`                                             | 1) subtract<br/>2) negative (prefix)   | yes<br/>no |  left<br/>right   |
|                                             `*`                                             | multiply                               |    yes     |       left        |
|                                             `/`                                             | divide                                 |    yes     |       left        |
|                                             `%`                                             | modulo                                 |    yes     |       left        |
|                                             `~`                                             | power                                  |    yes     |       left        |
|                                            `>>`                                             | right bit-shift                        |    yes     |       left        |
|                                            `<<`                                             | left bit-shift                         |    yes     |       left        |
|                                             `&`                                             | 1) bit-wise _AND_<br/>2) boolean _AND_ |    yes     |       left        |
|                                       <code>\|</code>                                       | 1) bit-wise _OR_<br/>2) boolean _OR_   |    yes     |       left        |
|                                             `^`                                             | 1) bit-wise _XOR_<br/>2) boolean _XOR_ |    yes     |       left        |
| `=`, `+=`, `-=`, `*=`, `/=`,<br/>`~=`, `%=`, `<<=`, `>>=`, `&=`,<br/><code>\|=</code>, `^=` | assignments                            |    yes     |       right       |
|                                            `==`                                             | equals to                              |    yes     |       left        |
|                                            `~=`                                             | not equals to                          |    yes     |       left        |
|                                             `>`                                             | greater than                           |    yes     |       left        |
|                                            `>=`                                             | greater than or equals to              |    yes     |       left        |
|                                             `<`                                             | less than                              |    yes     |       left        |
|                                            `<=`                                             | less than or equals to                 |    yes     |       left        |
|                                            `&&`                                             | boolean _AND_ (short-circuits)         |    yes     |       left        |
|                                      <code>\|\|</code>                                      | boolean _OR_ (short-circuits)          |    yes     |       left        |
|                                             `!`                                             | boolean _NOT_                          |     no     |       left        |
|                                         `[` .. `]`                                          | indexing                               |    yes     |       right       |
|                                             `.`                                             | 1) property access<br/>2) method call  |    yes     |       right       |


Symbols and Patterns
--------------------

| Symbol                             |         Name         | Description                           |
| ---------------------------------- | :------------------: | ------------------------------------- |
| `_`                                |      underscore      | default `switch` case                 |
| `;`                                |      semicolon       | statement separator                   |
| `,`                                |        comma         | list separator                        |
| `:`                                |        colon         | [object map] property value separator |
| `::`                               |         path         | module path separator                 |
| `#{` .. `}`                        |       hash map       | [object map] literal                  |
| `"` .. `"`                         |     double quote     | [string]                              |
| `'` .. `'`                         |     single quote     | [character][string]                   |
| `\`                                |        escape        | escape character literal              |
| `(` .. `)`                         |     parentheses      | expression grouping                   |
| `{` .. `}`                         |        braces        | block statement                       |
| <code>\|</code> .. <code>\|</code> |        pipes         | closure                               |
| `[` .. `]`                         |       brackets       | [array] literal                       |
| `!`                                |         bang         | function call in calling scope        |
| `=>`                               |     double arrow     | `switch` expression case separator    |
| `//`                               |       comment        | line comment                          |
| `/*` .. `*/`                       |       comment        | block comment                         |
| `(*` .. `*)`                       |       comment        | _reserved_                            |
| `<` .. `>`                         |   angular brackets   | _reserved_                            |
| `++`                               |      increment       | _reserved_                            |
| `--`                               |      decrement       | _reserved_                            |
| `..`                               |        range         | _reserved_                            |
| `...`                              |        range         | _reserved_                            |
| `**`                               |    exponentiation    | _reserved_                            |
| `#`                                |         hash         | _reserved_                            |
| `@`                                |          at          | _reserved_                            |
| `$`                                |        dollar        | _reserved_                            |
| `->`                               |        arrow         | _reserved_                            |
| `<-`                               |      left arrow      | _reserved_                            |
| `===`                              |   strict equals to   | _reserved_                            |
| `!==`                              | strict not equals to | _reserved_                            |
| `:=`                               |      assignment      | _reserved_                            |
| `::<` .. `>`                       |      turbofish       | _reserved_                            |
