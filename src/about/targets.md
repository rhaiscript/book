Supported Targets and Builds
===========================

{{#include ../links.md}}

Rhai supports all CPU and O/S targets supported by Rust, including:

* WebAssembly ([WASM])

* [`no-std`]

```admonish warning "32-bit big endian not supported"

Due to the [`smartstring`](https://crates.io/crates/smartstring) crate, 32-bit big endian
CPU architectures (e.g. PowerPC) are not supported.
```

Minimum Rust Version
--------------------

The minimum version of Rust required to compile Rhai is `1.51`.
