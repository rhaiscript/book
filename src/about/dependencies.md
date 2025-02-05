Dependencies
============

{{#include ../links.md}}

Rhai takes care to pull in as few dependencies as possible in order to avoid bloat when using the library.


Main Dependencies
-----------------

| Crate                                               | Description                                                   | Why use it?                                                                                                                                                                      |
| --------------------------------------------------- | ------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [`smallvec`](https://crates.io/crates/smallvec)     | `Vec` variant that stores a number of items inline            | most functions have very few parameters, and avoiding allocations result in significant performance improvement                                                                  |
| [`thin-vec`](https://crates.io/crates/thin-vec)     | `Vec` variant that takes up only the size of a single pointer | make key `enum` types with variants that contain `Vec`'s smaller                                                                                                                 |
| [`num-traits`](https://crates.io/crates/num-traits) | numeric traits                                                | for use with macros defining arithmetic functions and operators                                                                                                                  |
| [`ahash`](https://crates.io/crates/ahash)           | fast hashing for data                                         | not cryptographically secure, thus faster than standard Rust hashing; Rhai does a _lot_ of hashing so this matters                                                               |
| [`once_cell`](https://crates.io/crates/once_cell)   | global static data                                            | fixed hashing keys for [static hashing]                                                                                                                                          |
| [`bitflags`](https://crates.io/crates/bitflags)     | bit fields                                                    | store flags in [`AST`] nodes to minimize memory usage                                                                                                                            |
| [`smartstring`]                                     | `String` variant that stores short [strings] inline           | most [strings] in scripts (e.g. keywords, properties, symbols, variables, function names etc.) are short, and avoiding allocations result in significant performance improvement |


`no-std` Dependencies
---------------------

| Crate                                                        | Description                         |
| ------------------------------------------------------------ | ----------------------------------- |
| [`no-std-compat`](https://gitlab.com/jD91mZM2/no-std-compat) | create `std` imports under `no-std` |
| [`libm`](https://crates.io/crates/libm)                      | `no-std` math library               |
| [`core-error`](https://crates.io/crates/core-error)          | `Error` trait for `no-std`          |
| [`hashbrown`](https://crates.io/crates/hashbrown)            | `HashMap` and `HashSet`for `no-std` |


Feature Dependencies
--------------------

| Crate                                                 |  Pulled in by feature   |
| ----------------------------------------------------- | :---------------------: |
| [`rust_decimal`][rust_decimal]                        |       [`decimal`]       |
| [`unicode-xid`](https://crates.io/crates/unicode-xid) |  [`unicode-xid-ident`]  |
| [`serde`](https://crates.io/crates/serde)             | [`serde`], [`metadata`] |
| [`serde_json`](https://crates.io/crates/serde_json)   |      [`metadata`]       |
| [`rustyline`](https://crates.io/crates/rustyline)     |     `bin-features`      |


WASM Dependencies
-----------------

| Crate                                                   |     Pulled in by feature     |
| ------------------------------------------------------- | :--------------------------: |
| [`wasm-bindgen`](https://crates.io/crates/wasm-bindgen) |       [`wasm-bindgen`]       |
| [`stdweb`](https://crates.io/crates/stdweb)             |          [`stdweb`]          |
| [`instant`](https://crates.io/crates/instant)           | [`wasm-bindgen`], [`stdweb`] |
