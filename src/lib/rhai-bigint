`rhai-bigint`: Arbitrary-Precision BigInt Arithmetic
====================================================

{{#include ../links.md}}

`rhai-bigint` is an independent Rhai [package] that provides seamless, arbitrary-precision BigInt
arithmetic via the [`num-bigint`](https://crates.io/crates/num-bigint) crate.

```admonish info.side "Documentation"

See [https://docs.rs/rhai-bigint](https://docs.rs/rhai-bigint#api) for the list of functions.
```

> On `crates.io`: [`rhai-bigint`](https://crates.io/crates/rhai-bigint)
>
> On `GitHub`: [`rhaiscript/rhai-bigint`](https://github.com/rhaiscript/rhai-bigint)
>
> Package name: `BigIntPackage`


Dependency
----------

`Cargo.toml`:

```toml
[dependencies]
rhai = "{{version}}"
rhai-bigint = "0.1"             # use rhai-bigint crate
```


Load Package into [`Engine`]
----------------------------

```rust
use rhai::Engine;
use rhai::packages::Package;    // needed for 'Package' trait
use rhai_bigint::BigIntPackage;

let mut engine = Engine::new();

// Create new 'BigIntPackage' instance
let bigint = BigIntPackage::new();

// Load the package into the `Engine`
bigint.register_into_engine(&mut engine);
```


Features
--------

| Feature | Description                           | Default? |
| :-----: | ------------------------------------- | :------: |
| `sync`  | enables [`rhai/sync`][`sync`] support |  **no**  |
