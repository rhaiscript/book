Caveats and Gotcha's
====================

{{#include ../links.md}}

Rhai Grain is powerful, but a few caveats remain.


```admonish warning "Residual AST Fragments"
The transpiler does not promise that every construct can be proessed.

If there are still residual unsupported [`AST`] fragments, the resulting bytecodes
is not fully serializable.

That is not an error; it merely means the script still contains pieces that
Rhai Grain must fall back to the original [`AST`] interpreter.

Once `residual_count()` reaches zero, the [bytecodes] is completely stand-alone
and free of any dependency on the original source or [`AST`].
```

~~~admonish warning "`Engine` Compatibility"
The Rhai Grain [bytecodes] is not a portable binary format across arbitrary Rhai versions
or [feature][features] sets. It is intended to be produced and consumed by compatible builds of Rhai.

The target [`Engine`] must register all necessary functions, [modules] or custom behavior
that are called by the loaded script.

In other words, the [bytecodes] contains the script's logic, not a full copy of the host runtime.

A `Vm` can run a script with a fresh [`Engine`], but it must still know how to call
the functions the script expects.
~~~

```admonish warning "Stripped Diagnostics"
If a stripped [bytecodes] run fails at runtime, it can only tell which instructions
were active when it failed.

For full source locations, the [`Sidecar`] produced by `write_stripped()` must be preserved
and resolve the fault frames against it on the host with the script source.

Without the [`Sidecar`], the error can have no line or source mapping information.
```

```admonish note "Debugging Markers are Not Shipped by Default"
A [debugging][debugger] build can pause on statements and emit trace information.

That extra metadata is intentionally omitted from production [bytecodes].

This keeps the [bytecodes] smaller and avoids shipping a debugger into a deployment target.

The [bytecodes] still runs without those markers, but it cannot be stopped step-by-step
or at break-points in the same way as a full Rhai [`Engine`].
```

~~~admonish danger "Not a Replacement for `Engine` on the Target"
Rhai Grain does not remove the need for an [`Engine`] on the target.

It removes the need for a parser and an [`AST`], but the target still needs a
`Vm` and the runtime metadata that Rhai uses to call functions and manage values.

Rhai Grain is best thought of as a compact, deployable script representation,
not as an alternative runtime environment with zero host requirements.
~~~
