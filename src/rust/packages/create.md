Create a Custom Package
=======================

{{#include ../../links.md}}


The macro `def_package!` can be used to create a custom [package].

A custom package can aggregate many other packages into a single self-contained unit.
More functions can be added on top of others.

Custom packages are extremely useful when multiple [raw `Engine`] instances must be created such
that they all share the same set of functions.

For an example, see the [_One Engine Instance Per Call_]({{rootUrl}}/patterns/parallel.md) pattern.


`def_package!`
--------------

> `def_package!(`_root_`:`_name_`:"`_description_`", `_variable_`, { `_block_` });`

where:

|  Parameter  | Description                                                                                     |
| :---------: | ----------------------------------------------------------------------------------------------- |
|    root     | root namespace, usually `rhai`                                                                  |
|    name     | name of the package, usually ending in ...`Package`                                             |
| description | doc-comment for the package                                                                     |
|  variable   | a variable name holding a reference to the [module] (`&mut Module`) that is to form the package |
|    block    | a code block that initializes the package                                                       |


Examples
--------

```rust no_run
// Import necessary types and traits.
use rhai::{
    def_package,            // 'def_package!' macro
    packages::{             // pre-defined packages
        ArithmeticPackage, BasicArrayPackage, BasicMapPackage, LogicPackage
    }
};

// Define the package 'MyPackage'.
def_package!(rhai:MyPackage:"My own personal super package", module, {
    // Aggregate other packages simply by calling 'init' on each.
    ArithmeticPackage::init(module);
    LogicPackage::init(module);
    BasicArrayPackage::init(module);
    BasicMapPackage::init(module);

    // Register additional Rust functions using 'Module::set_native_fn'.
    let hash = module.set_native_fn("foo", |s: ImmutableString| {
        Ok(foo(s.into_owned()))
    });

    // Remember to update the parameter names/types and return type metadata
    // when using the 'metadata' feature.
    // 'Module::set_native_fn' by default does not set function metadata.
    module.update_fn_metadata(hash, &["s: ImmutableString", "i64"]);
});
```


Create a Custom Package from a Plugin Module
-------------------------------------------

By far the easiest way to create a custom module is to call `plugin::combine_with_exported_module!`
from within `def_package!` which simply merges in all the functions defined within a [plugin module].

In fact, this exactly is how Rhai's built-in packages, such as `BasicMathPackage`, are implemented.

Due to specific requirements of a [package], `plugin::combine_with_exported_module!`
_flattens_ all sub-modules (i.e. all functions and [type iterators] defined within sub-modules
are pulled up to the top level instead) and so there will not be any sub-modules added to the package.

Variables in the [plugin module] are ignored.

```rust no_run
// Import necessary types and traits.
use rhai::{
    def_package,
    packages::{ArithmeticPackage, BasicArrayPackage, BasicMapPackage, LogicPackage}
};
use rhai::plugin::*;

// Define plugin module.
#[export_module]
mod my_module {
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

// Define the package 'MyPackage'.
def_package!(rhai:MyPackage:"My own personal super package", module, {
    // Aggregate other packages simply by calling 'init' on each.
    ArithmeticPackage::init(module);
    LogicPackage::init(module);
    BasicArrayPackage::init(module);
    BasicMapPackage::init(module);

    // Merge all registered functions and constants from the plugin module into the custom package.
    //
    // The sub-module 'my_sub_module' is flattened and its functions registered at the top level.
    //
    // The text string name in the second parameter can be anything and is reserved for future use;
    // it is recommended to be an ID string that uniquely identifies the plugin module.
    //
    // The constant variable, 'MY_NUMBER', is ignored.
    //
    // This call ends up registering three functions at the top level of the package:
    //   1) greet
    //   2) get_num
    //   3) get_sub_num (pulled up from 'my_sub_module')
    //
    combine_with_exported_module!(module, "my-functions", my_module));
});
```


Create a Custom Package as an Independent Crate
----------------------------------------------

The project [`rhai-rand`](https://rhaiscript/rhai-rand) shows a simple example of creating a
custom [package] as an independent crate.

This allows the custom [package] to be used in multiple projects.

Essentially, the concept is to create a Rust crate that specifies [`rhai`](https://crates.io/crates/rhai) as dependency.
The main `lib.rs` module can contain the package being constructed.

```toml
┌────────────┐
│ Cargo.toml │
└────────────┘

[dependencies]
rhai = "{{version}}"    # assuming {{version}} is the latest version
```

```rust no_run
┌────────┐
│ lib.rs │
└────────┘

use rhai::def_package;
use rhai::plugin::*;

#[export_module]
mod my_module {
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
def_package!(rhai:MyPackage:"My own personal super package in a new crate!", module, {
    combine_with_exported_module!(module, "my-functions", my_module));
});
```
