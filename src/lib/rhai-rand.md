`rhai-rand`: Random Number Generation, Shuffling and Sampling
=============================================================

{{#include ../links.md}}

`rhai-rand` is an independent Rhai [package] that provides:

* random number generation using the [`rand`](https://crates.io/crates/rand) crate
* [array] shuffling and sampling

```admonish info.side "Documentation"

See [https://docs.rs/rhai-rand](https://docs.rs/rhai-rand#api) for the list of functions.
```

> On `crates.io`: [`rhai-rand`](https://crates.io/crates/rhai-rand)
>
> On `GitHub`: [`rhaiscript/rhai-rand`](https://github.com/rhaiscript/rhai-rand)
>
> Package name: `RandomPackage`


Dependency
----------

`Cargo.toml`:

```toml
[dependencies]
rhai = "{{version}}"
rhai-rand = "0.1"       # use rhai-rand crate
```


Load Package into [`Engine`]
----------------------------

```rust
use rhai::Engine;
use rhai::packages::Package;    // needed for 'Package' trait
use rhai_rand::RandomPackage;

let mut engine = Engine::new();

// Create new 'RandomPackage' instance
let random = RandomPackage::new();

// Load the package into the `Engine`
random.register_into_engine(&mut engine);
```


Features
--------

|  Feature   | Description                                                  | Default? | Should not be used with Rhai feature |
| :--------: | ------------------------------------------------------------ | :------: | :----------------------------------: |
|  `float`   | enables random floating-point number generation              |   yes    |             [`no_float`]             |
|  `array`   | enables methods for [arrays]                                 |   yes    |             [`no_index`]             |
| `metadata` | enables [functions metadata] (turns on [`metadata`] in Rhai) |  **no**  |                                      |

~~~admonish example "Example: Working with `no_float` in Rhai"

`Cargo.toml`:

```toml
[dependencies]
# Rhai is set for 'no_float', meaning no floating-point support
rhai = { version="{{version}}", features = ["no_float"] }

# Use 'default-features = false' to clear defaults, then only add 'array'
rhai-rand = { version="0.1", default-features = false, features = ["array"] }
```
~~~
