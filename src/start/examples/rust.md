Rust Examples
============

{{#include ../../links.md}}

A number of examples can be found in the `examples` directory:

| Example                                                                         | Description                                                                                                                     |
| ------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| [`arrays_and_structs`]({{repoHome}}/examples/arrays_and_structs.rs)             | shows how to register a custom Rust type and using [arrays] on it                                                               |
| [`custom_types_and_methods`]({{repoHome}}/examples/custom_types_and_methods.rs) | shows how to register a custom Rust type and methods for it                                                                     |
| [`hello`]({{repoHome}}/examples/hello.rs)                                       | simple example that evaluates an expression and prints the result                                                               |
| [`reuse_scope`]({{repoHome}}/examples/reuse_scope.rs)                           | evaluates two pieces of code in separate runs, but using a common [`Scope`]                                                     |
| [`serde`]({{repoHome}}/examples/serde.rs)                                       | example to serialize and deserialize Rust types with [`serde`](https://crates.io/crates/serde) (requires the [`serde`] feature) |
| [`simple_fn`]({{repoHome}}/examples/simple_fn.rs)                               | shows how to register a simple function                                                                                         |
| [`strings`]({{repoHome}}/examples/strings.rs)                                   | shows different ways to register functions taking string arguments                                                              |
| [`threading`]({{repoHome}}/examples/threading.rs)                               | shows how to communication to an [`Engine`] running in a separate thread via an MPSC channel                                    |


Running Examples
----------------

Examples can be run with the following command:

```sh
cargo run --example {example_name}
```

`no-std` Samples
----------------

To illustrate `no-std` builds, a number of sample applications are available under the `no_std` directory:

| Sample                                           | Description                                                                                          | Optimization |                     Allocator                     | Panics |
| ------------------------------------------------ | ---------------------------------------------------------------------------------------------------- | :----------: | :-----------------------------------------------: | :----: |
| [`no_std_test`]({{repoHome}}/no_std/no_std_test) | bare-bones test application that evaluates a Rhai expression and sets the result as the return value |     size     | [`wee_alloc`](https://crates.io/crates/wee_alloc) | abort  |

`cargo run` cannot be used to run a `no-std` sample.  It must first be built:

```sh
cd no_std/no_std_test

cargo +nightly build --release

./target/release/no_std_test
```
