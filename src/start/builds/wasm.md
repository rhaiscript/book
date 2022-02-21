WebAssembly (WASM) Build
========================

{{#include ../../links.md}}

It is possible to use Rhai when compiling to WebAssembly (WASM).
This yields a scripting engine (and language) that can be run in a standard web browser.

Why you would _want_ to is another matter... as there is already a nice, fast, complete scripting language
for the the common WASM environment (i.e. a browser) &ndash; and it is called JavaScript.

But anyhow, do it because you _can_!

When building for WASM, certain features will not be available,
such as the script file API's and loading modules from external script files.

```admonish example.small "Sample"

Check out the [_Online Playground_]({{rootUrl}}/tools/playground.md) project which is driven
by a Rhai [`Engine`] compiled into [WASM].
```


JavaScript Interop
------------------

Specify either of the [`wasm-bindgen`] or [`stdweb`] features when building for WASM.
This selects the appropriate JavaScript interop layer to use.

It is still possible to compile for WASM without either [`wasm-bindgen`] or [`stdweb`],
but then the interop code must be explicitly provided.


Size
----

Also look into [minimal builds] to reduce generated WASM size.

As of this version, a typical, full-featured Rhai scripting engine compiles to a single WASM file
less than 200KB gzipped.

When excluding features that are marginal in WASM environment, the gzipped payload can be
further shrunk to 160KB.


Speed
-----

In benchmark tests, a WASM build runs scripts roughly 1.7-2.2x slower than a native optimized release build.


Common Features
---------------

Some Rhai functionalities are not necessary in a WASM environment, so the following features
are typically used for a WASM build:

|            Feature             | Description                                                                                                                                                                                                                           |
| :----------------------------: | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [`wasm-bindgen`] or [`stdweb`] | use [`wasm-bindgen`](https://crates.io/crates/wasm-bindgen) or [`stdweb`](https://crates.io/crates/stdweb) as the JavaScript interop layer, omit if using custom interop code                                                         |
|         [`unchecked`]          | when a WASM module panics, it doesn't crash the entire web app; however this also disables [maximum number of operations] and [progress] tracking so a script can still run indefinitely &ndash; the web app must terminate it itself |
|          [`only_i32`]          | WASM supports 32-bit and 64-bit integers, but most scripts will only need 32-bit                                                                                                                                                      |
|         [`f32_float`]          | WASM supports 32-bit single-precision and 64-bit double-precision floating-point numbers, but single-precision is usually fine for most uses                                                                                          |
|         [`no_module`]          | a WASM module cannot load modules from the file system, so usually this is not needed, but the savings are minimal; alternatively, a custom [module resolver] can be provided that loads other Rhai scripts                           |

The following features are typically _not_ used because they don't make sense in a WASM build:

|    Feature    | Why unnecessary                                                                                       |
| :-----------: | ----------------------------------------------------------------------------------------------------- |
|   [`sync`]    | WASM is single-threaded                                                                               |
|  [`no_std`]   | `std` lib works fine with WASM                                                                        |
| [`metadata`]  | WASM usually doesn't need access to Rhai functions metadata                                           |
| [`internals`] | WASM usually doesn't need to access Rhai internal data structures, unless you are walking the [`AST`] |
| [`debugging`] | unless debugging is needed                                                                            |
