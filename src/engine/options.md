Engine Configuration Options
============================

{{#include ../links.md}}

A number of other configuration options are available from the [`Engine`] to fine-tune behavior and safeguards.


Compile-Time Language Features
------------------------------

| Method                                                             | Description                                                                            |
| ------------------------------------------------------------------ | -------------------------------------------------------------------------------------- |
| `set_optimization_level`<br/>(not available under [`no_optimize`]) | sets the amount of script _optimizations_ performed (see [script optimization])        |
| `set_allow_if_expression`                                          | allows/disallows [`if`-expressions]({{rootUrl}}/language/if-expression.md)             |
| `set_allow_switch_expression`                                      | allows/disallows [`switch` expressions]({{rootUrl}}/language/switch-expression.md)     |
| `set_allow_statement_expression`                                   | allows/disallows [statement expressions]({{rootUrl}}/language/statement-expression.md) |
| `set_allow_anonymous_fn`<br/>(not available under [`no_function`]) | allows/disallows [anonymous functions]                                                 |
| `set_allow_looping`                                                | allows/disallows looping (i.e. [`while`], [`loop`], [`do`] and [`for`] statements)     |
| `set_allow_shadowing`                                              | allows/disallows _[shadowing]_ of [variables]                                          |
| `set_strict_variables`                                             | enables/disables [_Strict Variables_ mode][strict variables]                           |
| `disable_symbol`                                                   | disables a certain [keyword] or [operator] (see [disable keywords and operators])      |

Beware that these options activate during _compile-time_ only.  If an [`AST`] is compiled on an
[`Engine`] but then evaluated on a different [`Engine`] with different configuration, disallowed
features contained inside the [`AST`] will still run as normal.


Runtime Behavior
----------------

| Method                                                                     | Description                                                                                                      |
| -------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------- |
| `set_fail_on_invalid_map_property`<br/>(not available under [`no_object`]) | sets whether to raise errors (instead of returning [`()`]) when invalid properties are accessed on [object maps] |

Safety Limits
-------------

| Method                |     Not available under      | Description                                                                                                               |
| --------------------- | :--------------------------: | ------------------------------------------------------------------------------------------------------------------------- |
| `set_max_expr_depths` |        [`unchecked`]         | sets the maximum nesting levels of an expression/statement (see [maximum statement depth])                                |
| `set_max_call_levels` |        [`unchecked`]         | sets the maximum number of function call levels (default 50) to avoid infinite recursion (see [maximum call stack depth]) |
| `set_max_operations`  |        [`unchecked`]         | sets the maximum number of _operations_ that a script is allowed to consume (see [maximum number of operations])          |
| `set_max_modules`     |        [`unchecked`]         | sets the maximum number of [modules] that a script is allowed to load (see [maximum number of modules])                   |
| `set_max_string_size` |        [`unchecked`]         | sets the maximum length (in UTF-8 bytes) for [strings] (see [maximum length of strings])                                  |
| `set_max_array_size`  | [`unchecked`], [`no_index`]  | sets the maximum size for [arrays] (see [maximum size of arrays])                                                         |
| `set_max_map_size`    | [`unchecked`], [`no_object`] | sets the maximum number of properties for [object maps] (see [maximum size of object maps])                               |
