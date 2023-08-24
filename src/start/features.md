Optional Features
=================

{{#include ../links.md}}

By default, Rhai includes all the standard functionalities in a small, tight package.

```admonish warning "Features are not additive"

Most Rhai features are not strictly _additive_, i.e. they do not only add optional functionalities.

In fact, most features are _subtractive_, i.e. they opt-**out** of unneeded functionalities.
Notice that this deviates from Rust norm where features are _additive_.

Excluding functionalities result in smaller, faster builds as well as more control over
what scripts can (or cannot) do.

There is a reason for this design, because the _lack_ of a language feature by itself is a feature (that's deep...).

See [here]({{rootUrl}}/patterns/multiple.md) for more details.
```


Features that Enable Special Functionalities
--------------------------------------------

| Feature             | Additive? | Description                                                                                                                                                                                                                                                        |
| ------------------- | :-------: | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `std`               |  **no**   | standard features                                                                                                                                                                                                                                                  |
| `sync`              |  **no**   | restricts all values types to those that are `Send + Sync`; under this feature, all Rhai types, including [`Engine`], [`Scope`] and [`AST`], are all `Send + Sync`                                                                                                 |
| `decimal`           |  **no**   | enables the [`Decimal`][rust_decimal] number type (pulls in the [`rust_decimal`][rust_decimal] crate)                                                                                                                                                              |
| `unicode-xid-ident` |  **no**   | allows [Unicode Standard Annex #31](http://www.unicode.org/reports/tr31/) as identifiers (pulls in the [`unicode-xid`](https://crates.io/crates/unicode-xid) crate)                                                                                                |
| `serde`             |    yes    | enables serialization/deserialization via `serde` (pulls in the [`serde`](https://crates.io/crates/serde) crate)                                                                                                                                                   |
| `metadata`          |    yes    | enables exporting [functions metadata]; implies `serde` and additionally pulls in [`serde_json`](https://crates.io/crates/serde_json)                                                                                                                              |
| `internals`         |    yes    | exposes internal data structures (e.g. [`AST`] nodes);<br/><br/>**Safety Warnings**<ul><li>allowing access to internal types may enable external attack vectors</li><li>internal types and functions are volatile and may change from version to version</li></ul> |
| `debugging`         |    yes    | enables the [debugging][debugger] interface; implies `internals`                                                                                                                                                                                                   |


Features that Disable Certain Language Features
-----------------------------------------------

| Feature            | Additive? | Description                                               |
| ------------------ | :-------: | --------------------------------------------------------- |
| `no_float`         |  **no**   | disables floating-point numbers and math                  |
| `no_index`         |  **no**   | disables [arrays] and indexing features                   |
| `no_object`        |  **no**   | disables support for [custom types] and [object maps]     |
| `no_time`          |  **no**   | disables [timestamps]                                     |
| `no_function`      |  **no**   | disables script-defined [functions]; implies `no_closure` |
| `no_module`        |  **no**   | disables loading external [modules]                       |
| `no_closure`       |  **no**   | disables capturing external variables in [closures]       |
| `no_custom_syntax` |  **no**   | disables [custom syntax] and [custom operators]           |


Features that Disable Certain Engine Features
---------------------------------------------

| Feature       | Additive? | Description                                                                                                                                                                                                                                                                                                                                                   |
| ------------- | :-------: | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `unchecked`   |  **no**   | disables [arithmetic checking][checked] (such as over-flows and division by zero), [call stack depth limit][maximum call stack depth], [operations count limit][maximum number of operations], [modules loading limit][maximum number of modules] and [data size limit][maximum length of strings].<br/>Beware that a bad script may panic the entire system! |
| `no_optimize` |  **no**   | disables [script optimization]                                                                                                                                                                                                                                                                                                                                |
| `no_position` |  **no**   | disables position tracking during parsing                                                                                                                                                                                                                                                                                                                     |


Features that Configure the Engine
----------------------------------

| Feature     | Additive? | Description                                                                                         |
| ----------- | :-------: | --------------------------------------------------------------------------------------------------- |
| `f32_float` |  **no**   | sets the system floating-point type (`FLOAT`) to `f32` instead of `f64`; no effect under `no_float` |
| `only_i32`  |  **no**   | sets the system integer type (`INT`) to `i32` and disable all other integer types                   |
| `only_i64`  |  **no**   | sets the system integer type (`INT`) to `i64` and disable all other integer types                   |


Features for `no-std` Builds
----------------------------

The following features are provided exclusively for [`no-std`] targets.
Do not use them when not compiling for [`no-std`].

Specify `default-features = false` when compiling for [`no-std`], which will remove the default
`std` feature.

| Feature  | Additive? | Description                                                                                                    |
| -------- | :-------: | -------------------------------------------------------------------------------------------------------------- |
| `no_std` |  **no**   | builds for [`no-std`]; notice that additional dependencies will be pulled in to replace missing `std` features |


Features for WebAssembly (WASM) Builds
--------------------------------------

The following features are provided exclusively for [WASM] targets.
Do not use them for non-[WASM] targets.

| Feature        | Additive? | Description                                                                        |
| -------------- | :-------: | ---------------------------------------------------------------------------------- |
| `wasm-bindgen` |  **no**   | uses [`wasm-bindgen`](https://crates.io/crates/wasm-bindgen) to compile for [WASM] |
| `stdweb`       |  **no**   | uses [`stdweb`](https://crates.io/crates/stdweb) to compile for [WASM]             |


Features for Building Bin Tools
-------------------------------

The feature `bin-features` include all the features necessary for building the [bin tools](bin.md).

By default, it includes: `decimal`, `metadata`, `serde`, `debugging` and `rustyline`.


Example
-------

The `Cargo.toml` configuration below:

```toml
[dependencies]
rhai = { version = "{{version}}", features = [ "sync", "unchecked", "only_i32", "no_float", "no_module", "no_function" ] }
```

turns on these six features:

|    Feature    | Description                                                                          |
| :-----------: | ------------------------------------------------------------------------------------ |
|    `sync`     | everything is `Send + Sync`                                                          |
|  `unchecked`  | disable all [safety checks][safety] (should not be used with untrusted user scripts) |
|  `only_i32`   | use only 32-bit signed integers and no others                                        |
|  `no_float`   | no floating point numbers                                                            |
|  `no_module`  | no loading external [modules]                                                        |
| `no_function` | no defining [functions]                                                              |

The resulting scripting engine supports only the `i32` integer numeral type (and no others like
`u32`, `i16` or `i64`), no floating-point, is `Send + Sync` (so it can be safely used across
threads), and does not support defining [functions] nor loading external [modules].

This configuration is perfect for an expression parser in a 32-bit embedded system without
floating-point hardware.
