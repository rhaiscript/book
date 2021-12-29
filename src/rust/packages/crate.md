Create a Custom Package as an Independent Crate
==============================================

{{#include ../../links.md}}

The project [`rhai-rand`](https://rhaiscript/rhai-rand) shows a simple example of creating a
custom [package] as an independent crate.

This allows the custom [package] to be used in multiple projects.

Essentially, the concept is to create a Rust crate that specifies
[`rhai`](https://crates.io/crates/rhai) as dependency.
The main `lib.rs` module can contain the [package] being constructed.

```toml
┌────────────┐
│ Cargo.toml │
└────────────┘

[package]
name = "my-package"     # 'my-package' crate

[dependencies]
rhai = "{{version}}"          # assuming {{version}} is the latest version
```

```rust no_run
┌────────┐
│ lib.rs │
└────────┘

use rhai::def_package;
use rhai::plugin::*;

// This is a plugin module
#[export_module]
mod my_module {
    // Constants are ignored when used as a package
    pub const MY_NUMBER: i64 = 42;

    pub fn greet(name: &str) -> String {
        format!("hello, {}!", name)
    }
    pub fn get_num() -> i64 {
        42
    }

    // This is a sub-module, but if using combine_with_exported_module!, it will
    // be flattened and all functions registered at the top level.
    pub mod my_sub_module {
        pub fn get_sub_num() -> i64 {
            0
        }
    }
}

// Define the package 'MyPackage' which is exported for the crate.
def_package! {
    /// My own personal super package in a new crate!
    rhai:MyPackage => |module| {
        combine_with_exported_module!(module, "my-functions", my_module));
    }
}
```
