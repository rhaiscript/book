`rhai-fs`: Random Number Generation, Shuffling and Sampling
=============================================================

{{#include ../links.md}}

`rhai-fs` is an independent Rhai [package] that provides functions to read from and write to files.

> On `crates.io`: [`rhai-fs`](https://crates.io/crates/rhai-fs)
>
> On `GitHub`: [`rhaiscript/rhai-fs`](https://github.com/rhaiscript/rhai-fs)
>
> Package name: `FilesystemPackage`


Dependency
----------

`Cargo.toml`:

```toml
[dependencies]
rhai = "{{version}}"
rhai-fs = "0.1"       # use rhai-fs crate
```


Load Package into [`Engine`]
----------------------------

```rust
use rhai::Engine;
use rhai::packages::Package;    // needed for 'Package' trait
use rhai_fs::FilesystemPackage;

let mut engine = Engine::new();

// Create new 'FilesystemPackage' instance
let random = FilesystemPackage::new();

// Load the package into the `Engine`
random.register_into_engine(&mut engine);
```


Example
-------

```js
// Create a file, or open it if already exists
let file = open_file("example.txt");

// Read the contents of the file (if any) into a BLOB
let blob_buf = file.read_blob();

print(`file contents: ${blob_buf}`);

// Update BLOB data
blob_buf.write_utf8(0..=0x20, "foobar");

print(`new file contents: ${blob_buf}`);

// Seek back to the beginning
file.seek(0);

// Overwrite the original file with new data
file.write(blob_buf);
```


Features
--------

|  Feature   | Description                                                  | Default? | Should be used with Rhai feature |
| :--------: | ------------------------------------------------------------ | :------: | :------------------------------: |
| `no_array` | removes support for [arrays] and [BLOB's]                    |    no    |           [`no_index`]           |
| `metadata` | enables [functions metadata] (turns on [`metadata`] in Rhai) |    no    |                                  |
