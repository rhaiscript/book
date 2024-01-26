`rhai-ml`: Functions for AI and Machine Learning
================================================

{{#include ../links.md}}


`rhai-ml` is an independent Rhai [package] that provides functions useful for
artificial intelligence and machine learning.

```admonish info.side "Documentation"

See [https://docs.rs/rhai-ml](https://docs.rs/rhai-ml#api) for the list of functions.
```

> On `crates.io`: [`rhai-ml`](https://crates.io/crates/rhai-ml)
>
> On `GitHub`: [`rhaiscript/rhai-ml`](https://github.com/rhaiscript/rhai-ml)
>
> Package name: `MLPackage`


Dependency
----------

`Cargo.toml`:

```toml
[dependencies]
rhai = "{{version}}"
rhai-ml = "0.1"       # use rhai-ml crate
```


Features
--------

|  Feature   | Description                                                                                   | Default? |
| :--------: | --------------------------------------------------------------------------------------------- | :------: |
| `metadata` | enables [functions metadata] (turns on [`metadata`] in Rhai); necessary for running doc-tests |  **no**  |


Load Package into [`Engine`]
----------------------------

```rust
use rhai::Engine;
use rhai::packages::Package;    // needed for 'Package' trait
use rhai_ml::MLPackage;

let mut engine = Engine::new();

// Create new 'MLPackage' instance
let ml = MLPackage::new();

// Load the package into the [`Engine`]
ml.register_into_engine(&mut engine);
```
