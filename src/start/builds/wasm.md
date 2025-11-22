WebAssembly (WASM) Build
========================

{{#include ../../links.md}}

```admonish question.side.wide "But why?"

There is already a fast and powerful scripting language that integrates nicely with WASM &ndash; **JavaScript**.

Anyhow, do it because you _can_!
```

It is possible to use Rhai when compiling to WebAssembly (WASM).

This yields a scripting engine (and language) that can be run in a standard web browser,
among other places.

```admonish warning "Unavailable features"

When building for WASM, certain features will not be available,
such as the script file APIs and loading [modules] from external script files.
```

```admonish example "Sample"

Check out the [_Online Playground_]({{rootUrl}}/tools/playground.md) project which is driven
by a Rhai [`Engine`] compiled into WASM.
```


JavaScript Interop
------------------

Specify [`wasm-bindgen`] when building for WASM that requires interop with JavaScript.
This selects [`wasm-bindgen`] as the JavaScript interop layer to use.

It is still possible to compile for WASM without [`wasm-bindgen`], but then the interop code must
then be explicitly provided.


Target Environments
-------------------

~~~admonish abstract "WASI: `wasm32-wasi`"

There is no particular setting to tweak when building for WASI.
~~~

~~~admonish abstract "JavaScript: `wasm32-unknown-unknown` + `wasm-bindgen`"

Rhai requires a system-provided source of random numbers (for hashing).

Such random number source is available from JavaScript (implied by `wasm-bindgen`).

The `js` feature on the [`getrandom`](https://crates.io/crates/getrandom) crate is
enabled automatically to provide the random  number source.
See also: <https://docs.rs/getrandom/latest/getrandom/#webassembly-support> for details.
~~~

~~~admonish warning "Raw: `wasm32-unknown-unknown`"

Rhai requires a system-provided source of random numbers (for hashing).

Non-JavaScript/non-browser environments may not have random numbers available, so it is necessary to
opt out of `default-features` in order to enable [static hashing] which uses fixed (non-random) keys.

```toml
[dependencies]
rhai = { version = "{{version}}", default-features = false, features = [ "std" ] }
```
~~~


Size
----

Also look into [minimal builds] to reduce generated WASM size.

A typical, full-featured Rhai scripting engine compiles to a single WASM32 file that is less than
400KB (non-gzipped).

When excluding features that are marginal in WASM environment, the gzipped payload can be shrunk further.

Standard [packages][built-in packages] can also be excluded to yield additional size savings.


Speed
-----

In benchmark tests, a WASM build runs scripts roughly 30% slower than a native optimized release build.


Common Features
---------------

Some Rhai functionalities are not necessary in a WASM environment, so the following features
are typically used for a WASM build:

|       Feature        | Description                                                                                                                                                                                                                           |
| :------------------: | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
|   [`wasm-bindgen`]   | use [`wasm-bindgen`](https://crates.io/crates/wasm-bindgen) as the JavaScript interop layer, omit if using custom interop code                                                                                                        |
|    [`unchecked`]     | when a WASM module panics, it doesn't crash the entire web app; however this also disables [maximum number of operations] and [progress] tracking so a script can still run indefinitely &ndash; the web app must terminate it itself |
|     [`only_i32`]     | WASM supports 32-bit and 64-bit integers, but most scripts will only need 32-bit                                                                                                                                                      |
|    [`f32_float`]     | WASM supports 32-bit single-precision and 64-bit double-precision floating-point numbers, but single-precision is usually fine for most uses                                                                                          |
|    [`no_module`]     | a WASM module cannot load modules from the file system, so usually this is not needed, but the savings are minimal; alternatively, a custom [module resolver] can be provided that loads other Rhai scripts                           |
| [`no_custom_syntax`] | if [custom syntax] is not used, this results in a small size saving                                                                                                                                                                   |

The following features are typically _not_ used because they don't make sense in a WASM build:

|    Feature    | Why unnecessary                                                                                       |
| :-----------: | ----------------------------------------------------------------------------------------------------- |
|   [`sync`]    | WASM is single-threaded                                                                               |
|  [`no_std`]   | `std` lib works fine with WASM                                                                        |
| [`metadata`]  | WASM usually doesn't need access to Rhai functions metadata                                           |
| [`internals`] | WASM usually doesn't need to access Rhai internal data structures, unless you are walking the [`AST`] |
| [`debugging`] | unless debugging is needed                                                                            |
