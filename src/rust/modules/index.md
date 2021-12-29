Modules
=======

{{#include ../../links.md}}

Rhai allows organizing functionalities (functions, both Rust-based or script-based, and variables)
into independent _modules_.

Modules can be disabled via the [`no_module`] feature.

A module has the type `Module` and holds a collection of functions, variables,
[type iterators] and sub-modules.

It may be created entirely from Rust functions, or it may encapsulate a Rhai script together
with the functions and variables defined by that script.

Other scripts can then load this module and use the functions and variables exported
as if they were defined inside the same script.

Alternatively, modules can be registered directly into an [`Engine`] and made available
to scripts either globally or under individual static module [_namespaces_][function namespaces].


Usage Patterns
--------------

| Usage          |                API                |          Lookup          | Sub-modules? | Variables? |
| -------------- | :-------------------------------: | :----------------------: | :----------: | :--------: |
| Global module  | `Engine:: register_global_module` |       simple name        |   ignored    |  ignored   |
| Static module  | `Engine:: register_static_module` | namespace-qualified name |     yes      |    yes     |
| Dynamic module |       [`import`] statement        | namespace-qualified name |     yes      |    yes     |
