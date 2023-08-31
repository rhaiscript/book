Export Functions Metadata to JSON
=================================

{{#include ../../links.md}}


`Engine::gen_fn_metadata_to_json`<br/>`Engine::gen_fn_metadata_with_ast_to_json`
--------------------------------------------------------------------------------

As part of a _reflections_ API, `Engine::gen_fn_metadata_to_json` and the corresponding
`Engine::gen_fn_metadata_with_ast_to_json` export the full list of [custom types] and
[functions metadata] in JSON format.

~~~admonish warning.small "Requires `metadata`"

The [`metadata`] feature is required for this API, which also pulls in the
[`serde_json`](https://crates.io/crates/serde_json) crate.
~~~

### Sources

Functions and [custom types] from the following sources are included:

1. Script-defined functions in an [`AST`] (for `Engine::gen_fn_metadata_with_ast_to_json`)
2. Native Rust functions registered into the global namespace via the `Engine::register_XXX` API
3. [Custom types] registered into the global namespace via the `Engine::register_type_with_name` API
4. _Public_ (i.e. non-[`private`]) functions (native Rust or Rhai scripted) and [custom types] in static modules
   registered via `Engine::register_static_module`
5. Native Rust functions and [custom types] in external [packages] registered via `Engine::register_global_module`
6. Native Rust functions and [custom types] in [built-in packages] (optional)


JSON Schema
-----------

The JSON schema used to hold metadata is very simple, containing a nested structure of
`modules`, a list of `customTypes` and a list of `functions`.

### Module Schema

```json
{
    "doc": "//! Module documentation",

    "modules":
    {
        "sub_module_1": /* namespace 'sub_module_1' */
        {
            "modules":
            {
                "sub_sub_module_A": /* namespace 'sub_module_1::sub_sub_module_A' */
                {
                    "doc": "//! Module documentation can also occur in any sub-module",

                    "customTypes": /* custom types exported in 'sub_module_1::sub_sub_module_A' */
                    [
                        { ... custom type metadata ... },
                        { ... custom type metadata ... },
                        { ... custom type metadata ... }
                        ...
                    ],
                    "functions": /* functions exported in 'sub_module_1::sub_sub_module_A' */
                    [
                        { ... function metadata ... },
                        { ... function metadata ... },
                        { ... function metadata ... },
                        { ... function metadata ... }
                        ...
                    ]
                },
                "sub_sub_module_B": /* namespace 'sub_module_1::sub_sub_module_B' */
                {
                    ...
                }
            }
        },
        "sub_module_2": /* namespace 'sub_module_2' */
        {
            ...
        },
        ...
    },

    "customTypes": /* custom types registered globally */
    [
        { ... custom type metadata ... },
        { ... custom type metadata ... },
        { ... custom type metadata ... },
        ...
    ],

    "functions": /* functions registered globally or in the 'AST' */
    [
        { ... function metadata ... },
        { ... function metadata ... },
        { ... function metadata ... },
        { ... function metadata ... },
        ...
    ]
}
```

### Custom Type Metadata Schema

```json
{
    "typeName": "alloc::string::String",    /* name of Rust type */
    "displayName": "MyType",
    "docComments":  /* omitted if none */
    [
        "/// My super-string type.",
        ...
    ]
}
```

### Function Metadata Schema

```json
{
    "baseHash": 9876543210,     /* partial hash with only number of parameters */
    "fullHash": 1234567890,     /* full hash with actual parameter types */
    "namespace": "internal" | "global",
    "access": "public" | "private",
    "name": "fn_name",
    "isAnonymous": false,
    "type": "native" | "script",
    "numParams": 42,            /* number of parameters */
    "params":                   /* omitted if no parameters */
    [
        { "name": "param_1", "type": "type_1" },
        { "name": "param_2" },  /* no type name */
        { "type": "type_3" },   /* no parameter name */
        ...
    ],
    "thisType": "this_type",    /* omitted if none */
    "returnType": "ret_type",   /* omitted if () or unknown */
    "signature": "[private] fn_name(param_1: type_1, param_2, _: type_3) -> ret_type",
    "docComments":              /* omitted if none */
    [
        "/// doc-comment line 1",
        "/// doc-comment line 2",
        "/** doc-comment block */",
        ...
    ]
}
```
