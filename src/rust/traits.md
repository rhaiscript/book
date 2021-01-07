Traits
======

{{#include ../links.md}}

A number of traits, under the `rhai::` module namespace, provide additional functionalities.

| Trait                    | Description                                                        | Methods                                                               |
| ------------------------ | ------------------------------------------------------------------ | --------------------------------------------------------------------- |
| `RegisterFn`             | trait for registering functions                                    | `register_fn`                                                         |
| `RegisterResultFn`       | trait for registering [fallible functions]                         | `register_result_fn`                                                  |
| `Func`                   | trait for creating Rust closures from script                       | `create_from_ast`, `create_from_script`                               |
| `ModuleResolver`         | trait implemented by [module resolution][module resolver] services | `resolve`                                                             |
| `plugin::PluginFunction` | trait implemented by [plugin] functions                            | `call`, `is_method_call`, `is_variadic`, `clone_boxed`, `input_types` |
