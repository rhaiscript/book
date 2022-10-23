Built-in Module Resolvers
=========================

{{#include ../../../links.md}}

There are a number of standard [module resolvers] built into Rhai, the default being the
[`FileModuleResolver`](file.md) which simply loads a script file based on the path (with `.rhai`
extension attached) and execute it to form a [module].

Built-in [module resolvers] are grouped under the `rhai::module_resolvers` module.
