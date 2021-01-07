Rust Examples
============

{{#include ../../links.md}}

A number of examples can be found in the `examples` directory:

| Example                                                                         | Description                                                                                                                                  |
| ------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------- |
| [`arrays_and_structs`]({{repoTree}}/examples/arrays_and_structs.rs)             | shows how to register a custom Rust type and using [arrays] on it                                                                            |
| [`custom_types_and_methods`]({{repoTree}}/examples/custom_types_and_methods.rs) | shows how to register a custom Rust type and methods for it                                                                                  |
| [`hello`]({{repoTree}}/examples/hello.rs)                                       | simple example that evaluates an expression and prints the result                                                                            |
| [`reuse_scope`]({{repoTree}}/examples/reuse_scope.rs)                           | evaluates two pieces of code in separate runs, but using a common [`Scope`]                                                                  |
| [`serde`]({{repoTree}}/examples/serde.rs)                                       | example to serialize and deserialize Rust types with [`serde`](https://crates.io/crates/serde).<br/>The [`serde`] feature is required to run |
| [`simple_fn`]({{repoTree}}/examples/simple_fn.rs)                               | shows how to register a simple function                                                                                                      |
| [`strings`]({{repoTree}}/examples/strings.rs)                                   | shows different ways to register functions taking string arguments                                                                           |


Running Examples
----------------

Examples can be run with the following command:

```bash
cargo run --example {example_name}
```

`no-std` Samples
----------------

To illustrate `no-std` builds, a number of sample applications are available under the `no_std` directory:

| Sample                                           | Description                                                                                          | Optimization |                     Allocator                     | Panics |
| ------------------------------------------------ | ---------------------------------------------------------------------------------------------------- | :----------: | :-----------------------------------------------: | :----: |
| [`no_std_test`]({{repoTree}}/no_std/no_std_test) | bare-bones test application that evaluates a Rhai expression and sets the result as the return value |     size     | [`wee_alloc`](https://crates.io/crates/wee_alloc) | abort  |

`cargo run` cannot be used to run a `no-std` sample.  It must first be built:

```bash
cd no_std/no_std_test

cargo +nightly build --release

./target/release/no_std_test
```
