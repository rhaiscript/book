Capabilities
============

{{#include ../links.md}}


Why is This Important?
----------------------

Normally, there is no way for Rhai's [`AST`] to go out of sync with the [`Engine`] build that
compiled it.

The Rhai compiler and its runtime are tightly-coupled together, and an [`AST`] can only ever be
built from the script source.  There is no mechanism to store an [`AST`] externally or transmit it
elsewhere.

Not so for the Rhai Grain VM.

A Rhai script's [`AST`] can be further transpiled into [bytecodes] which can be stored externally
(i.e. into a file) and then shipped to another machine, and run on a completely different [`Engine`]
build, which may not even have a parser or the same [features] enabled.


Fingerprint (ABI)
-----------------

Thereforel, Rhai Grain [bytecodes] carry a small _fingerprint_, or ABI, that makes sure it can be
properly decoded and/or run on the host.


### Fingerprint (ABI) structure

```admonish note.side
The fingerprint is not just metadata.

It is the compatibility guard-rails that prevents a [bytecodes] stream from running
on a build that does not have the features to handle it.
```

The fingerprint records:

- whether the [system integer number][standard types] is 32-bit or 64-bit,

- whether the system [floating-point number][standard types] is 32-bit or 64-bit,

- the _capabilities_ required to run.


### Capabilities set

The capability set covers Rhai language features that a script may use, such as:

- floating-point or [`decimal`] numbers,

- [arrays], [BLOB's], or [object maps],

- indexing (i.e. usage of `[ .. ]` syntax),

- accessing [properties][getters/setters],

- calling functions in [method-call][method] style,

- the `this` pointer,

- defining scripted [functions],

- [function pointers] and [currying],

- capturing variables from [closures],

- [`import`] and [`export`] to work with [modules],

- [custom syntax]


What is Checked?
----------------

When `Program::read` loads [bytecodes], it checks the ABI before it decodes the rest,
comparing the capabilities it requires against the capabilities supported by the host.

If any required capability is missing from the host, it simply fails to load.

`Program::read` will never load [bytecodes] beyond the current host's ability to handle.

This catches two classes of incompatibility:

1. a different integer or floating-point width, such as using `i64` on a host built with
   [`only_i32`] or using `f32` on a host built without [`f32_float`], and

2. any required capability that the host does not support.


### Example

- If the [bytecodes] was transpiled on a Rhai instance with a 64-bit integer build, but is being loaded
  on a 32-bit build of Rhai (via [`only_i32`]), it will refuse to load.

- If the [bytecodes] was transpiled on a full Rhai instance, for example, but it does not use any
  [arrays] or [object maps].  Then the same [bytecodes] can be successfully loaded and run on a
  computer with a [`no_index`] and [`no_object`] build of Rhai, because those features are not used.

- If the [bytecodes] uses a particular language feature of Rhai (e.g. defines a [function]) but the
  host build does not support that feature (e.g. [`no_function`]), again it will refuse to load.

- The key point is that Rhai Grain treats a [bytecodes] stream as a deployment unit: if it requires a
  capability the host build does not support, it is not runnable.


Verification
------------

The ABI check above is only the first gate.

After the [bytecodes] is decoded, a verifier inspects every instruction checks the capabilities that
instruction requires to run.

This is a second, more precise check. Even if the fingerprint is declared to be compatible with the
host, the [bytecodes] stream itself is inspected to make sure it does not contain instructions
that require additional capabilities.

In other words, Rhai Grain enforces that:

- [bytecodes] capabilities are within the host's supported capabilities, and

- every instruction's requirements fall under the declared capabilities.

Only when both hold can will the [bytecodes] safely load.


Caveats and Gotcha's
--------------------

Rhai Grain's capability system is for compatibility, not a general-purpose sandbox.

It is designed to make sure that a stream of [bytecodes] can run on the current host build.

It does not make the [bytecodes] fully portable to every possible build of Rhai.

- A build with more [features] is usually fine; a build with fewer features is not.
  The host must support every capability the [bytecodes] require.

- This check happens before loading and execution. A mismatch fails at load time,
  not as a later runtime fault.

- The compatibility test is intentionally strict. `Program::read` will fail even if
  the host's integer or floating-point widths are longer than the ones declared
  by the [bytecodes].

- This is not a substitute for runtime [safety] limits. Even fully compatible
  [bytecodes] still requires usual Rhai runtime checks.
