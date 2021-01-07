Modules
=======

{{#include ../../links.md}}

Rhai allows organizing code (functions, both Rust-based or script-based, and variables) into _modules_.
Modules can be disabled via the [`no_module`] feature.

A module is of the type `Module` and holds a collection of functions, variables, [type iterators] and sub-modules.
It may be created entirely from Rust functions, or it may encapsulate a Rhai script together with the functions
and variables defined by that script.

Other scripts can then load this module and use the functions and variables exported
as if they were defined inside the same script.
