Side-Effect Considerations for Full Optimization Level
======================================================

{{#include ../../links.md}}

All of Rhai's built-in functions (and operators which are implemented as functions) are _pure_
(i.e. they do not mutate state nor cause any side-effects, with the exception of `print` and `debug`
which are handled specially) so using [`OptimizationLevel::Full`] is usually quite safe _unless_
custom types and functions are registered.

If custom functions are registered, they _may_ be called (or maybe not, if the calls happen to lie
within a pruned code block).

If custom functions are registered to overload [built-in operators], they will also be called when
the operators are used (in an [`if`] statement, for example), potentially causing side-effects.

```admonish tip.small "Rule of thumb"

* _Always_ register custom types and functions _after_ compiling scripts if [`OptimizationLevel::Full`] is used.

* _DO NOT_ depend on knowledge that the functions have no side-effects, because those functions can change later on and,
  when that happens, existing scripts may break in subtle ways.
```
