Keywords
========

{{#include ../links.md}}

The following are reserved keywords in Rhai.

| Active keywords                                                            | Reserved keywords                                          | Usage                               |     Inactive under feature     |
| -------------------------------------------------------------------------- | ---------------------------------------------------------- | ----------------------------------- | :----------------------------: |
| `true`, `false`                                                            |                                                            | [constants]                         |                                |
| [`let`][variable], [`const`][constant]                                     | `var`, `static`                                            | [variables]                         |                                |
| `is_shared`                                                                |                                                            | _shared_ values                     |         [`no_closure`]         |
|                                                                            | `is`                                                       | type checking                       |                                |
| [`if`], [`else`][`if`]                                                     | `goto`, `exit`                                             | control flow                        |                                |
| [`switch`]                                                                 | `match`, `case`                                            | switching and matching              |                                |
| [`do`], [`while`], [`loop`], `until`, [`for`], [`in`], `continue`, `break` |                                                            | looping                             |                                |
| [`fn`][function], [`private`], `is_def_fn`, `this`                         | `public`, `protected`, `new`                               | [functions]                         |        [`no_function`]         |
| [`return`]                                                                 |                                                            | return values                       |                                |
| [`throw`], [`try`], [`catch`]                                              |                                                            | [throw/catch][`catch`] [exceptions] |                                |
| [`import`], [`export`], `as`                                               | `use`, `with`, `module`, `package`, `super`                | [modules]                           |         [`no_module`]          |
| [`global`]                                                                 |                                                            | automatic global [module]           | [`no_function`], [`no_module`] |
| [`Fn`][function pointer], `call`, [`curry`][currying]                      |                                                            | [function pointers]                 |                                |
|                                                                            | `spawn`, `thread`, `go`, `sync`, `async`, `await`, `yield` | threading/async                     |                                |
| [`type_of`], [`print`], [`debug`], [`eval`], `is_def_var`                  |                                                            | special functions                   |                                |
|                                                                            | `default`, `void`, `null`, `nil`                           | special values                      |                                |

```admonish warning
Keywords cannot become the name of a [function] or [variable], even when they are
[disabled][disable keywords and operators].
```
