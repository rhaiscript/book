Rust Examples
============

{{#include ../../links.md}}


Standard Examples
-----------------

A number of examples can be found under `examples`.

| Example                                                                         | Description                                                                                                                     |
| ------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| [`arrays_and_structs`]({{repoHome}}/examples/arrays_and_structs.rs)             | shows how to register a [Rust type][custom type] and using it with [arrays]                                                     |
| [`callback`](callback.rs)                                                       | shows how to store a Rhai [closure] and call it later within Rust                                                               |
| [`custom_types_and_methods`]({{repoHome}}/examples/custom_types_and_methods.rs) | shows how to register a [Rust type][custom type] and methods/getters/setters for it                                             |
| [`hello`]({{repoHome}}/examples/hello.rs)                                       | simple example that evaluates an expression and prints the result                                                               |
| [`reuse_scope`]({{repoHome}}/examples/reuse_scope.rs)                           | evaluates two pieces of code in separate runs, but using a common [`Scope`]                                                     |
| [`serde`]({{repoHome}}/examples/serde.rs)                                       | example to serialize and deserialize Rust types with [`serde`](https://crates.io/crates/serde) (requires the [`serde`] feature) |
| [`simple_fn`]({{repoHome}}/examples/simple_fn.rs)                               | shows how to register a simple Rust function                                                                                    |
| [`strings`]({{repoHome}}/examples/strings.rs)                                   | shows different ways to register Rust functions taking [string] arguments                                                       |
| [`threading`]({{repoHome}}/examples/threading.rs)                               | shows how to communicate with an [`Engine`] running in a separate thread via an MPSC channel                                    |


Scriptable Event Handler With State Examples
-------------------------------------------

Because of its popularity, the pattern [_Scriptable Event Handler With State_]({{rootUrl}}/patterns/events.md)
has sample implementations for different styles.

| Example                                                          | Handler Script                                                                           |                   Description                    |
| ---------------------------------------------------------------- | ---------------------------------------------------------------------------------------- | :----------------------------------------------: |
| [`event_handler_main`]({{repoHome}}/examples/event_handler_main) | [`event_handler_main/script.rhai`]({{repoHome}}/examples/event_handler_main/script.rhai) | [_Main Style_]({{rootUrl}}/patterns/events-1.md) |
| [`event_handler_js`]({{repoHome}}/examples/event_handler_js)     | [`event_handler_js/script.rhai`]({{repoHome}}/examples/event_handler_js/script.rhai)     |  [_JS Style_]({{rootUrl}}/patterns/events-2.md)  |
| [`event_handler_map`]({{repoHome}}/examples/event_handler_map)   | [`event_handler_map/script.rhai`]({{repoHome}}/examples/event_handler_map/script.rhai)   | [_Map Style_]({{rootUrl}}/patterns/events-3.md)  |


Running Examples
----------------

Examples can be run with the following command:

```sh
cargo run --example {example_name}
```

`no-std` Examples
-----------------

To illustrate `no-std` builds, a number of example applications are available under the `no_std` directory:

| Example                                          | Description                                                                                          | Optimization |                     Allocator                     | Panics |
| ------------------------------------------------ | ---------------------------------------------------------------------------------------------------- | :----------: | :-----------------------------------------------: | :----: |
| [`no_std_test`]({{repoHome}}/no_std/no_std_test) | bare-bones test application that evaluates a Rhai expression and sets the result as the return value |     size     | [`wee_alloc`](https://crates.io/crates/wee_alloc) | abort  |


### Building the `no-std` examples

```admonish warning "Nightly required"

Currently, the nightly compiler must be used to build for `no-std`.
```

```sh
cd no_std/no_std_test

cargo +nightly build --release

./target/release/no_std_test
```
