Keywords
========

The following are reserved keywords in Rhai.

| Active keywords                                                                                                            | Reserved keywords                                          | Usage                                       |
| -------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------- | ------------------------------------------- |
| `true`, `false`                                                                                                            |                                                            | constants                                   |
| [`let`](variable.md), [`const`](constant.md)                                                                               | `var`, `static`                                            | variables                                   |
| `is_shared`                                                                                                                |                                                            | _shared_ values                             |
|                                                                                                                            | `is`                                                       | type checking                               |
| [`if`](if.md), [`else`](if.md)                                                                                             | `goto`                                                     | control flow                                |
| [`switch`](switch.md)                                                                                                      | `match`, `case`                                            | switching and matching                      |
| [`do`](do.md), [`while`](while.md), [`loop`](loop.md), `until`, [`for`](for.md), [`in`](operators.md), `continue`, `break` |                                                            | looping                                     |
| [`fn`](functions.md), [`private`](modules/export.md), `is_def_fn`, `this`                                                  | `public`, `protected`, `new`                               | [functions](functions.md)                   |
| [`return`](return.md)                                                                                                      |                                                            | return values                               |
| [`throw`](throw.md), [`try`](try-catch.md), [`catch`](try-catch.md)                                                        |                                                            | [throw/catch](try-catch.md) exceptions      |
| [`import`](modules/import.md), [`export`](modules/export.md), `as`                                                         | `use`, `with`, `module`, `package`, `super`                | [modules](modules/index.md)                 |
| [`global`](global.md)                                                                                                      |                                                            | automatic global [module](modules/index.md) |
| [`Fn`](fn-ptr.md), `call`, [`curry`](fn-curry.md)                                                                          |                                                            | [function pointers](fn-ptr.md)              |
|                                                                                                                            | `spawn`, `thread`, `go`, `sync`, `async`, `await`, `yield` | threading/async                             |
| [`type_of`](type-of.md), [`print`](print-debug.md), [`debug`](print-debug.md), [`eval`](eval.md), `is_def_var`             |                                                            | special functions                           |
|                                                                                                                            | `default`, `void`, `null`, `nil`                           | special values                              |

```admonish warning.small
Keywords cannot become the name of a function or variable, even when they are disabled.
```
