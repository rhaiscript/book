Keywords List
=============

{{#include ../links.md}}

|        Keyword        | Description                                                             |         Inactive under          | Is function? |
| :-------------------: | ----------------------------------------------------------------------- | :-----------------------------: | :----------: |
|        `true`         | boolean true literal                                                    |                                 |    **no**    |
|        `false`        | boolean false literal                                                   |                                 |    **no**    |
|         `let`         | [variable] declaration                                                  |                                 |    **no**    |
|        `const`        | [constant] declaration                                                  |                                 |    **no**    |
|         `if`          | [`if`] statement                                                        |                                 |    **no**    |
|        `else`         | `else` block of [`if`] statement                                        |                                 |    **no**    |
|       `switch`        | [matching][`switch`]                                                    |                                 |    **no**    |
|         `do`          | [looping][`do`]                                                         |                                 |    **no**    |
|        `while`        | <ol><li>[`while`] loop</li><li>condition for [`do`] loop</li></ol>      |                                 |    **no**    |
|        `until`        | [`do`] loop                                                             |                                 |    **no**    |
|        `loop`         | infinite [`loop`]                                                       |                                 |    **no**    |
|         `for`         | [`for`] loop                                                            |                                 |    **no**    |
|         `in`          | <ol><li>[containment][`in`] test</li><li>part of [`for`] loop</li></ol> |                                 |    **no**    |
|         `!in`         | negated [containment][`in`] test                                        |                                 |    **no**    |
|      `continue`       | continue a loop at the next iteration                                   |                                 |    **no**    |
|        `break`        | break out of loop iteration                                             |                                 |    **no**    |
|       `return`        | [return][`return`] value                                                |                                 |    **no**    |
|        `throw`        | throw [exception]                                                       |                                 |    **no**    |
|         `try`         | [trap][`try`] [exception]                                               |                                 |    **no**    |
|        `catch`        | [catch][`catch`] [exception]                                            |                                 |    **no**    |
|       `import`        | [import][`import`] [module]                                             |          [`no_module`]          |    **no**    |
|       `export`        | [export][`export`] [variable]                                           |          [`no_module`]          |    **no**    |
|         `as`          | alias for [variable] [export][`export`]                                 |          [`no_module`]          |    **no**    |
|       `global`        | automatic [global][`global`] [module]                                   | [`no_module`], [`no_function`]  |    **no**    |
|       `private`       | mark [function] [private][`private`]                                    |         [`no_function`]         |    **no**    |
| `fn` (lower-case `f`) | [function] definition                                                   |         [`no_function`]         |    **no**    |
|  `Fn` (capital `F`)   | create a [function pointer]                                             |                                 |     yes      |
|        `call`         | call a [function pointer]                                               |                                 |     yes      |
|        `curry`        | curry a [function pointer]                                              |                                 |     yes      |
|      `is_shared`      | is a [variable] shared?                                                 | [`no_function`], [`no_closure`] |     yes      |
|      `is_def_fn`      | is [function] defined?                                                  |         [`no_function`]         |     yes      |
|     `is_def_var`      | is [variable] defined?                                                  |                                 |     yes      |
|        `this`         | reference to base object for method call                                |         [`no_function`]         |    **no**    |
|       `type_of`       | get type name of value                                                  |                                 |     yes      |
|        `print`        | print value                                                             |                                 |     yes      |
|        `debug`        | print value in debug format                                             |                                 |     yes      |
|        `eval`         | evaluate script                                                         |                                 |     yes      |


Reserved Keywords
-----------------

|   Keyword   | Potential usage       |
| :---------: | --------------------- |
|    `var`    | variable declaration  |
|  `static`   | variable declaration  |
|  `shared`   | share value           |
|   `goto`    | control flow          |
|   `exit`    | control flow          |
|   `match`   | matching              |
|   `case`    | matching              |
|  `public`   | function/field access |
| `protected` | function/field access |
|    `new`    | constructor           |
|    `use`    | import namespace      |
|   `with`    | scope                 |
|    `is`     | type check            |
|  `module`   | module                |
|  `package`  | package               |
|   `super`   | base class/module     |
|  `thread`   | threading             |
|   `spawn`   | threading             |
|    `go`     | threading             |
|   `await`   | async                 |
|   `async`   | async                 |
|   `sync`    | async                 |
|   `yield`   | async                 |
|  `default`  | special value         |
|   `void`    | special value         |
|   `null`    | special value         |
|    `nil`    | special value         |
