Modules
=======

Rhai allows organizing code into _modules_.

A module holds a collection of [functions](../functions.md), [constants](../constants.md) and sub-modules.

It may encapsulates a Rhai script together with the [functions](../functions.md) and
[constants](../constants.md) defined by that script.

Other scripts can then load this module and use the [functions](../functions.md) and
[constants](../constants.md) exported as if they were defined inside the same script.
