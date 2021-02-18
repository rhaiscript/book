Keywords
========

{{#include ../links.md}}

The following are reserved keywords in Rhai:

| Active keywords                                                  | Reserved keywords                                          | Usage                    | Inactive under feature |
| ---------------------------------------------------------------- | ---------------------------------------------------------- | ------------------------ | :--------------------: |
| `true`, `false`                                                  |                                                            | [constants]              |                        |
| `let`, `const`                                                   | `var`, `static`                                            | [variables]              |                        |
|                                                                  | `begin`, `end`                                             | block scopes             |                        |
| `is_shared`                                                      |                                                            | _shared_ values          |     [`no_closure`]     |
| `if`, `else`                                                     | `then`, `unless`, `goto`, `exit`                           | control flow             |                        |
| `switch`                                                         | `match`, `case`                                            | switching and matching   |                        |
| `do`, `while`, `loop`, `until`, `for`, `in`, `continue`, `break` | `each`                                                     | looping                  |                        |
| `fn`, `private`                                                  | `public`, `new`                                            | [functions]              |    [`no_function`]     |
| `return`                                                         |                                                            | return values            |                        |
| `throw`, `try`, `catch`                                          |                                                            | throw/catch [exceptions] |                        |
| `import`, `export`, `as`                                         | `use`, `with`, `module`, `package`                         | [modules]                |     [`no_module`]      |
| `Fn`, `call`, `curry`                                            |                                                            | [function pointers]      |                        |
|                                                                  | `spawn`, `thread`, `go`, `sync`, `async`, `await`, `yield` | threading/async          |                        |
| `type_of`, `print`, `debug`, `eval`                              |                                                            | special functions        |                        |
|                                                                  | `default`, `void`, `null`, `nil`                           | special values           |                        |

Keywords cannot become the name of a [function] or [variable], even when they are disabled.
