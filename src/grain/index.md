Rhai Grain
==========

{{#include ../links.md}}

Rhai Grain is an _experimental_ [bytecodes][bytecodes-wiki] transpiler and VM for Rhai scripts.

It is only available under the [`grain`] feature.

See [documentation on Rhai Grain](https://docs.rs/rhai/latest/rhai/grain/index.html).


When to Use Rhai Grain
----------------------

```admonish warning.small "Warning: Experimental"

Rhai Grain is still experimental and is not yet fully production-ready.

```

Use Rhai Grain when you want to:

- ship a script without shipping source code,

- reduce per-script memory and CPU costs on constrained devices,

- run faster than the standard Rhai interpreter,

- avoid keeping an [`AST`] resident,

- run the same script on a second Rhai instance without reparsing it.


Use the normal [`AST`] for the most straight-forward developer experience, including
[debugging][debugger], when the script source is available, or when the script
is short-lived and compiled just once in-process.

Rhai Grain is most valuable when a script is treated as an artifact: transpiled into
[bytecodes][bytecodes-wiki] once, serialized once, validated once and then executed many times
directly on a target machine.


Caveats
-------

- `Program::write` only succeeds when `Program::residual_count` is zero.

- Anything that fails to transpile is kept as an [`AST`] fragment and interpreted by Rhai.

- `Program::write_stripped` is for shipping [bytecodes] plus a host-side [`Sidecar`]
  for diagnostics.


Unsupported Features
--------------------

Rhai Grain does not support every feature of the language.

When the transpiler failes, it preserves that part as a residual [`AST`] fragment and hands it back to
Rhai for processing.

They include:

- [`eval`],

- loading [modules] via [`import`],

- the [`Engine::on_map_missing_property`]({{rootUrl}}/language/object-maps-missing-prop.md) advanced callback,

- [custom syntax]

If `Program::residual_count()` is non-zero, the program still runs, but part of it will fall back to
Rhai's normal [`AST`] evaluation path.
