`rhai-rand`: Random Number Generation, Shuffling and Sampling
=============================================================

{{#include ../links.md}}

`rhai-rand` is an independent Rhai [package] that provides:

* random number generation using the [`rand`](https://crates.io/crates/rand) crate
* [array] shuffling and sampling

On `crates.io`: [`rhai-rand`](https://crates.io/crates/rhai-rand)

On `GitHub`: [`rhaiscript/rhai-rand`](https://github.com/rhaiscript/rhai-rand)

Package name: `RandomPackage`


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
use rhai::packages::Package;    // needed for 'as_shared_module'
use rhai_rand::RandomPackage;

let mut engine = Engine::new();

// Create new 'RandomPackage' instance
let random = RandomPackage::new();

// Load the package
engine.register_global_module(random.as_shared_module());
```


Features
--------

|  Feature   | Description                                                  | Default? | Should not be used with Rhai feature |
| :--------: | ------------------------------------------------------------ | :------: | :----------------------------------: |
|  `float`   | enables random floating-point number generation              |   yes    |             [`no_float`]             |
|  `array`   | enables methods for [arrays]                                 |   yes    |             [`no_index`]             |
| `metadata` | enables [functions metadata] (turns on [`metadata`] in Rhai) |    no    |                                      |

### Example &ndash; working with `no_float` in Rhai

`Cargo.toml`:

```toml
[dependencies]
# Rhai is set for 'no_float', meaning no floating-point support
rhai = { version="{{version}}", features = ["no_float"] }

# Use 'default-features = false' to clear defaults, then only add 'array'
rhai-rand = { version="0.1", default-features = false, features = ["array"] }
```


Package Functions
-----------------

The following functions are defined.

|      Function       | Return type | Feature | Description                                                                  |
| :-----------------: | :---------: | :-----: | ---------------------------------------------------------------------------- |
|      `rand()`       |    `INT`    |         | generates a random number                                                    |
| `rand(start..end)`  |    `INT`    |         | generates a random number within the exclusive range `start..end`            |
| `rand(start..=end)` |    `INT`    |         | generates a random number within the inclusive range `start..=end`           |
|   `rand_float()`    |   `FLOAT`   | `float` | generates a random floating-point number between `0.0` and `1.0` (exclusive) |
|    `rand_bool()`    |   `bool`    |         | generates a random boolean                                                   |
|   `rand_bool(p)`    |   `bool`    | `float` | generates a random boolean with the probability `p` of being `true`          |


### Arrays

The following methods are defined for [arrays] (requires the `array` feature).

|  Method   |                       Parameter(s)                        | Return type | Description                                                                |
| :-------: | :-------------------------------------------------------: | :---------: | -------------------------------------------------------------------------- |
| `shuffle` |                          _none_                           |             | shuffles the items in the [array]                                          |
| `sample`  |                          _none_                           | [`Dynamic`] | returns a random item from the [array]                                     |
| `sample`  | number of items to sample (empty if ≤ 0, all if ≥ length) |  [`Array`]  | returns a non-repeating _shuffled_ random sample of items from the [array] |
