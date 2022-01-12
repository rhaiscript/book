Create a Custom Package
=======================

{{#include ../../links.md}}

The macro `def_package!` can be used to create a custom [package].

A custom [package] can aggregate many other [packages] into a single self-contained unit.
More functions can be added on top of others.

Custom [packages] are extremely useful when multiple [raw `Engine`] instances must be created such
that they all share the same set of functions.

For an example, see the [_One Engine Instance Per Call_]({{rootUrl}}/patterns/parallel.md) pattern.


`def_package!`
--------------

> ```rust,no_run
> def_package! {
>     /// Package description doc-comment
>     root::name => |variable| {
>                         :
>         // package initialization code block
>                         :
>     }
>
>     // Multiple packages can be defined at the same time
>
>     /// Package description doc-comment
>     root::name => |variable| {
>                         :
>         // package initialization code block
>                         :
>     }
> 
>     :
> }
> ```

where:

|  Parameter  | Description                                                                                             |
| :---------: | ------------------------------------------------------------------------------------------------------- |
| description | doc-comment for the package                                                                             |
|    root     | root namespace, usually `rhai`                                                                          |
|    name     | name of the package, usually ending in ...`Package`                                                     |
|  variable   | a variable name holding a reference to the [module] forming the package, usually `module`, `m` or `lib` |
| code block  | a code block that initializes the package                                                               |


Examples
--------

```rust,no_run
// Import necessary types and traits.
use rhai::def_package;      // 'def_package!' macro
use rhai::packages::{
    ArithmeticPackage, BasicArrayPackage, BasicMapPackage, LogicPackage
};

def_package! {
    /// My own personal super package
    rhai::MyPackage => |module| {
        // Aggregate other packages simply by calling 'init' on each.
        ArithmeticPackage::init(module);
        LogicPackage::init(module);
        BasicArrayPackage::init(module);
        BasicMapPackage::init(module);

        // Register additional Rust functions using 'Module::set_native_fn'.
        let hash = module.set_native_fn("foo", |s: &str| Ok(foo(s)));

        // Remember to update the parameter names/types and return type
        // metadata when using the 'metadata' feature because
        // 'Module::set_native_fn' by default does not set function metadata.
        module.update_fn_metadata(hash, &["s: &str", "i64"]);
    }
}
```


Create a Custom Package from a Plugin Module
-------------------------------------------

By far the easiest way to create a custom [package] is to call `plugin::combine_with_exported_module!`
from within `def_package!` which simply merges in all the functions defined within a [plugin module].

In fact, this exactly is how Rhai's built-in [packages], such as `BasicMathPackage`, are implemented.

Due to specific requirements of a [package], `plugin::combine_with_exported_module!`
_flattens_ all sub-modules (i.e. all functions and [type iterators] defined within sub-modules
are pulled up to the top level instead) and so there will not be any sub-modules added to the [package].

Variables in the [plugin module] are ignored.

```rust,no_run
// Import necessary types and traits.
use rhai::def_package;
use rhai::packages::{
    ArithmeticPackage, BasicArrayPackage, BasicMapPackage, LogicPackage
};
use rhai::plugin::*;

// Define plugin module.
#[export_module]
mod my_plugin_module {
    pub const MY_NUMBER: i64 = 42;

    pub fn greet(name: &str) -> String {
        format!("hello, {}!", name)
    }

    // Non-public functions are by default not exported.
    fn get_private_num() -> i64 {
        42
    }

    pub fn get_num() -> i64 {
        get_private_num()
    }

    // This is a sub-module, but if using 'combine_with_exported_module!',
    // it will be flattened and all functions registered at the top level.
    //
    // Because of the flattening, sub-modules are very convenient for
    // putting feature gates onto large groups of functions.
    #[cfg(feature = "sub-num-feature")]
    pub mod my_sub_module {
        pub fn get_sub_num() -> i64 {
            0
        }
    }
}

def_package! {
    /// My own personal super package
    rhai::MyPackage => |module| {
        // Aggregate other packages simply by calling 'init' on each.
        ArithmeticPackage::init(module);
        LogicPackage::init(module);
        BasicArrayPackage::init(module);
        BasicMapPackage::init(module);

        // Merge all registered functions and constants from the plugin module
        // into the custom package.
        //
        // The sub-module 'my_sub_module' is flattened and its functions
        // registered at the top level.
        //
        // The text string name in the second parameter can be anything
        // and is reserved for future use; it is recommended to be an
        // ID string that uniquely identifies the plugin module.
        //
        // The constant variable, 'MY_NUMBER', is ignored.
        //
        // This call ends up registering three functions at the top level of
        // the package:
        // 1) 'greet'
        // 2) 'get_num'
        // 3) 'get_sub_num' (flattened from 'my_sub_module')
        //
        combine_with_exported_module!(module, "my-mod", my_plugin_module));
    }
}
```
