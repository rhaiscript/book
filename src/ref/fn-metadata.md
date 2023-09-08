Functions Metadata
==================

{{#title Functions Metadata}}

The _metadata_ of a [function](functions.md) means all relevant information related to a
[function's](functions.md) definition including:

1. Its callable name

2. Its access mode (public or [private](modules/export.md))

3. Its parameter names (if any)

4. Its purpose, in the form of [doc-comments](comments.md)

5. Usage notes, warnings, examples etc., in the form of [doc-comments](comments.md)

A [function's](functions.md) _signature_ encapsulates the first three pieces of information in a
single concise line of definition:

> `[private]` _name_ `(`_param 1_`,` _param 2_`,` ... `,` _param n_ `)`


Get Functions Metadata
======================

The built-in [function](functions.md) `get_fn_metadata_list` returns an [array](arrays) of [object
maps](object-maps.md), each containing the metadata of one script-defined [function](functions.md)
in scope.

`get_fn_metadata_list` has a few versions taking different parameters:

| Signature                            | Description                                                                                                                                                      |
| ------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `get_fn_metadata_list()`             | returns an [array](arrays.md) for _all_ script-defined [functions](functions.md)                                                                                 |
| `get_fn_metadata_list(name)`         | returns an [array](arrays.md) containing all script-defined [functions](functions.md) matching a specified name                                                  |
| `get_fn_metadata_list(name, params)` | returns an [array](arrays.md) containing all script-defined [functions](functions.md) matching a specified name and accepting the specified number of parameters |

The return value is an [array](arrays.md) of [object maps](object-maps.md) containing the following fields.

| Field          |                       Type                        | Optional? | Description                                                                                           |
| -------------- | :-----------------------------------------------: | :-------: | ----------------------------------------------------------------------------------------------------- |
| `namespace`    |            [string](strings-chars.md)             |  **yes**  | the module _namespace_ if the [function](functions.md) is defined within a [module](modules/index.md) |
| `access`       |            [string](strings-chars.md)             |    no     | `"public"` if the function is public,<br/>`"private"` if it is [private](modules/export.md)           |
| `name`         |            [string](strings-chars.md)             |    no     | [function](functions.md) name                                                                         |
| `params`       | [array](arrays.md) of [strings](strings-chars.md) |    no     | parameter names                                                                                       |
| `this_type`    |            [string](strings-chars.md)             |  **yes**  | restrict the type of `this` if the [function](functions.md) is a [method](fn_methods.md)              |
| `is_anonymous` |                      `bool`                       |    no     | is this [function](functions.md) an [anonymous function](fn-anon.md)?                                 |
| `comments`     | [array](arrays.md) of [strings](strings-chars.md) |  **yes**  | [doc-comments](comments.md), if any, one per line                                                     |
