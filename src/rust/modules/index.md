Modules
=======

{{#include ../../links.md}}

Rhai allows organizing functionalities ([functions], both Rust-based and scripted, and
[variables]) into independent _modules_.

A module has the type `rhai::Module` and holds a collection of [functions], [variables], [type
iterators] and sub-modules.

It may contain entirely Rust functions, or it may encapsulate a Rhai script together with
all the [functions] and [variables] defined by that script.

Other scripts then load this module and use the [functions] and [variables] [exported][`export`].

Alternatively, modules can be registered directly into an [`Engine`] and made available to scripts,
either globally or under individual static module [_namespaces_][function namespaces].

Modules can be disabled via the [`no_module`] feature.


Usage Patterns
--------------

| Usage          |                API                |          Lookup          | Sub-modules? | Variables? |
| -------------- | :-------------------------------: | :----------------------: | :----------: | :--------: |
| Global module  | `Engine:: register_global_module` |       simple name        |   ignored    |  ignored   |
| Static module  | `Engine:: register_static_module` | namespace-qualified name |     yes      |    yes     |
| Dynamic module |       [`import`] statement        | namespace-qualified name |     yes      |    yes     |
