Use Rhai in Dynamic Libraries
=============================

{{#include ../links.md}}

Sometimes Rhai is pulled in as a dependency to build _dynamic libraries_ (a.k.a. _shared libraries_
in Linux or _DLL's_ in Windows), which are compiled separately from the main program, not linked
into the main binary, and loaded dynamically at runtime.


Problem Symptom
---------------

The _hasher_ used by Rhai, [`ahash`](https://crates.io/crates/ahash), automatically generates a
different _seed_ for hashing when compiling each library. This may create hash inconsistencies
between the main binary and the loaded dynamic library.

This usually happens when the [`Engine`] is passed into a function exposed by the dynamic library
and the [`Engine`]'s function registration API is called inside the dynamic library.

The symptom is usually _Function Not Found_ errors even though the relevant functions have already
been registered into the same [`Engine`].


Forcing a Stable Hashing Seed
-----------------------------

Specify `default-features = false` in `Cargo.toml` to force [`ahash`](https://crates.io/crates/ahash)
to use a stable seed instead of generating one for each compilation.

`Cargo.toml`:

```toml
[dependencies]
rhai = { version = "{{version}}", default-features = false }
```

```admonish warning.small "Warning: Safety considerations"

Using a stable seed allows dynamic libraries with Rhai code to be loaded and used with
an [`Engine`] in the main binary.

However, a predictable seed enlarges the attack surface of Rhai from malicious intent.
This safety trade-off should be carefully considered.
```
