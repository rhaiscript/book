`rhai-sci`: Functions for Scientific Computing
==============================================

{{#include ../links.md}}


`rhai-sci` is an independent Rhai [package] that provides functions useful for
scientific computing, inspired by languages like MATLAB, Octave, and R.

```admonish info.side "Documentation"

See [https://docs.rs/rhai-sci](https://docs.rs/rhai-sci#api) for the list of functions.
```

> On `crates.io`: [`rhai-sci`](https://crates.io/crates/rhai-sci)
>
> On `GitHub`: [`rhaiscript/rhai-sci`](https://github.com/rhaiscript/rhai-sci)
>
> Package name: `SciPackage`


Dependency
----------

`Cargo.toml`:

```toml
[dependencies]
rhai = "{{version}}"
rhai-sci = "0.1"       # use rhai-sci crate
```


Features
--------

|  Feature   | Description                                                                                                                                                                                                | Default? |
| :--------: | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :------: |
| `metadata` | enables [functions metadata] (turns on [`metadata`] in Rhai); necessary for running doc-tests                                                                                                              |  **no**  |
|    `io`    | enables the `read_matrix` function but pulls in several additional dependencies                                                                                                                            |   yes    |
| `nalgebra` | enables the functions `regress`, `inv`, `mtimes`, `horzcat`, `vertcat`, and `repmat` but pulls in [`nalgebra`](https://crates.io/crates/nalgebra) and [`linregress`](https://crates.io/crates/linregress). |   yes    |
|   `rand`   | enables the `rand` function for generating random values and random matrices, but pulls in [`rand`](https://crates.io/crates/rand).                                                                        |   yes    |


Load Package into [`Engine`]
----------------------------

```rust
use rhai::Engine;
use rhai::packages::Package;    // needed for 'Package' trait
use rhai_sci::SciPackage;

let mut engine = Engine::new();

// Create new 'SciPackage' instance
let sci = SciPackage::new();

// Load the package into the [`Engine`]
sci.register_into_engine(&mut engine);
```
