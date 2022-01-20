Get Functions Metadata in Scripts
================================

{{#include ../links.md}}

The built-in function `get_fn_metadata_list` returns an [array] of [object maps], each containing
the metadata of one script-defined [function] in scope.

`get_fn_metadata_list` has a few versions taking different parameters:

| Signature                            | Description                                                                                                                             |
| ------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------- |
| `get_fn_metadata_list()`             | returns an [array] for _all_ script-defined [functions]                                                                                 |
| `get_fn_metadata_list(name)`         | returns an [array] containing all script-defined [functions] matching a specified name                                                  |
| `get_fn_metadata_list(name, params)` | returns an [array] containing all script-defined [functions] matching a specified name and accepting the specified number of parameters |

[Functions] from the following sources are returned, in order:

1) Encapsulated script environment (e.g. when loading a [module] from a script file),
2) Current script,
3) [Modules] registered via `Engine::register_global_module` (latest registrations first)
4) [Modules] imported via the [`import`] statement (latest imports first),
5) [Modules] added via `Engine::register_static_module` (latest registrations first)

The return value is an [array] of [object maps] (so `get_fn_metadata_list` is also not available under
[`no_index`] or [`no_object`]), containing the following fields.

| Field          |         Type         | Optional? | Description                                                                         |
| -------------- | :------------------: | :-------: | ----------------------------------------------------------------------------------- |
| `namespace`    |       [string]       |    yes    | the module _namespace_ if the [function] is defined within a [module]               |
| `access`       |       [string]       |    no     | `"public"` if the function is public,<br/>`"private"` if it is [private][`private`] |
| `name`         |       [string]       |    no     | [function] name                                                                     |
| `params`       | [array] of [strings] |    no     | parameter names                                                                     |
| `is_anonymous` |        `bool`        |    no     | is this [function] an anonymous function?                                           |
