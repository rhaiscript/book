Keywords
========

{{#include ../links.md}}

The following are reserved keywords in Rhai:

| Active keywords                                                  | Reserved keywords                                          | Usage                    | Inactive under feature |
| ---------------------------------------------------------------- | ---------------------------------------------------------- | ------------------------ | :--------------------: |
| `true`, `false`                                                  |                                                            | [constants]              |                        |
| `let`, `const`, `global`                                         | `var`, `static`                                            | [variables]              |                        |
| `is_shared`                                                      |                                                            | _shared_ values          |     [`no_closure`]     |
| `if`, `else`                                                     | `goto`, `exit`                                             | control flow             |                        |
| `switch`                                                         | `match`, `case`                                            | switching and matching   |                        |
| `do`, `while`, `loop`, `until`, `for`, `in`, `continue`, `break` |                                                            | looping                  |                        |
| `fn`, `private`, `is_def_fn`, `this`                             | `public`, `protected`, `new`                               | [functions]              |    [`no_function`]     |
| `return`                                                         |                                                            | return values            |                        |
| `throw`, `try`, `catch`                                          |                                                            | throw/catch [exceptions] |                        |
| `import`, `export`, `as`                                         | `use`, `with`, `module`, `package`, `super`                | [modules]                |     [`no_module`]      |
| `Fn`, `call`, `curry`                                            |                                                            | [function pointers]      |                        |
|                                                                  | `spawn`, `thread`, `go`, `sync`, `async`, `await`, `yield` | threading/async          |                        |
| `type_of`, `print`, `debug`, `eval`, `is_def_var`                |                                                            | special functions        |                        |
|                                                                  | `default`, `void`, `null`, `nil`                           | special values           |                        |

Keywords cannot become the name of a [function] or [variable], even when they are disabled.
