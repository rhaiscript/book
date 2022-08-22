Create a Custom Package
=======================

{{#include ../../links.md}}

```admonish info.side "See also"

See also the [_One Engine Instance Per Call_]({{rootUrl}}/patterns/parallel.md) pattern.
```

The macro `def_package!` can be used to create a custom [package].

A custom [package] can aggregate many other [packages] into a single self-contained unit.
More functions can be added on top of others.

Custom [packages] are extremely useful when multiple [raw `Engine`] instances must be created such
that they all share the same set of functions.


`def_package!`
--------------

> ```rust
> def_package! {
>     /// Package description doc-comment
>     pub name(variable) {
>                         :
>         // package init code block
>                         :
>     }
>
>     // Multiple packages can be defined at the same time,
>     // possibly with base packages and/or code to setup an Engine.
>
>     /// Package description doc-comment
>     pub(crate) name(variable) : base_package_1, base_package_2, ... {
>                         :
>         // package init code block
>                         :
>     } |> |engine| {
>                         :
>         // engine setup code block
>                         :
>     }
> 
>     /// A private package description doc-comment
>     name(variable) {
>                         :
>         // private package init code block
>                         :
>     }
>
>     :
> }
> ```

where:

|         Element         | Description                                                                                          |
| :---------------------: | ---------------------------------------------------------------------------------------------------- |
|       description       | doc-comment for the [package]                                                                        |
|       `pub` etc.        | visibility of the [package]                                                                          |
|          name           | name of the [package], usually ending in ...`Package`                                                |
|        variable         | a variable name holding a reference to the [module] forming the [package], usually `module` or `lib` |
|      base_package       | an external [package] type that is merged into this [package] as a dependency                        |
| package init code block | a code block that initializes the [package]                                                          |
|         engine          | a variable name holding a mutable reference to an [`Engine`]                                         |
| engine setup code block | a code block that performs setup tasks on an [`Engine`] during registration                          |


Examples
--------

```rust
// Import necessary types and traits.
use rhai::def_package;      // 'def_package!' macro
use rhai::packages::{
    ArithmeticPackage, BasicArrayPackage, BasicMapPackage, LogicPackage
};

// Aggregate other base packages (if any) simply by listing them after a colon.
def_package! {
    /// My own personal super package
    pub MyPackage(module) : ArithmeticPackage, LogicPackage, BasicArrayPackage, BasicMapPackage
    {
        // Register additional Rust functions using 'Module::set_native_fn'.
        let hash = module.set_native_fn("foo", |s: &str| Ok(foo(s)));

        // Remember to update the parameter names/types and return type
        // metadata when using the 'metadata' feature because
        // 'Module::set_native_fn' by default does not set function metadata.
        module.update_fn_metadata(hash, &["s: &str", "i64"]);

        // Register a function for use as a custom operator.
        let hash = module.set_native_fn("@", |x: i64, y: i64| Ok(x * x + y * y));

        // Always make it available globally.
        module.update_fn_namespace(hash, FnNamespace::Global);
    } |> |engine| {
        // This optional block performs tasks on an 'Engine' instance,
        // e.g. register custom operators/syntax.

        // Define a custom operator '@' with precedence of 160
        // (i.e. between +|- and *|/).
        engine.register_custom_operator("@", 160).unwrap();
    }
}
```

~~~admonish tip.small "Tip: Feature gates on base packages"

Base packages in the list after the colon (`:`) can also have attributes (such as feature gates)!

```rust
def_package! {
    // 'BasicArrayPackage' is used only under 'arrays' feature.
    pub MyPackage(module) :
            ArithmeticPackage,
            LogicPackage,
            #[cfg(feature = "arrays")]
            BasicArrayPackage
    {
        ...
    }
}

```
~~~

~~~admonish danger.small "Advanced: `Engine` setup with `|>`"

A second code block (in the syntax of a [closure]) following a right-triangle symbol (`|>`)
is run whenever the [package] is being registered.

It allows performing setup tasks directly on that [`Engine`], e.g. registering [custom operators]
and/or [custom syntax].

```rust
def_package! {
    pub MyPackage(module) {
            :
            :
    } |> |engine| {
        // Call methods on 'engine'
    }
}
```
~~~


Create a Custom Package from a Plugin Module
--------------------------------------------

```admonish question.side "Trivia"

This is exactly how Rhai's built-in [packages], such as `BasicMathPackage`, are actually implemented.
```

By far the easiest way to create a custom [package] is to call `plugin::combine_with_exported_module!`
from within `def_package!` which simply merges in all the functions defined within a [plugin module].

Due to specific requirements of a [package], `plugin::combine_with_exported_module!`
_flattens_ all sub-modules (i.e. all functions and [type iterators] defined within sub-modules
are pulled up to the top level instead) and so there will not be any sub-modules added to the [package].

Variables in the [plugin module] are ignored.

```rust
// Import necessary types and traits.
use rhai::def_package;
use rhai::packages::{
    ArithmeticPackage, BasicArrayPackage, BasicMapPackage, LogicPackage
};
use rhai::plugin::*;

// Define plugin module.
#[export_module]
mod my_plugin_module {
    // Custom type.
    pub type ABC = TestStruct;

    // Public constant.
    pub const MY_NUMBER: i64 = 42;

    // Public function.
    pub fn greet(name: &str) -> String {
        format!("hello, {}!", name)
    }

    // Non-public functions are by default not exported.
    fn get_private_num() -> i64 {
        42
    }

    // Public function.
    pub fn get_num() -> i64 {
        get_private_num()
    }

    // Custom operator.
    #[rhai_fn(name = "@")]
    pub fn square_add(x: i64, y: i64) -> i64 {
        x * x + y * y
    }

    // A sub-module.  If using 'combine_with_exported_module!', however,
    // it will be flattened and all functions registered at the top level.
    //
    // Because of this flattening, sub-modules are very convenient for
    // putting feature gates onto large groups of functions.
    #[cfg(feature = "sub-num-feature")]
    pub mod my_sub_module {
        // Only available under 'sub-num-feature'.
        pub fn get_sub_num() -> i64 {
            0
        }
    }
}

def_package! {
    /// My own personal super package
    pub MyPackage(module) : ArithmeticPackage, LogicPackage, BasicArrayPackage, BasicMapPackage
    {
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
    } |> |engine| {
        // This optional block is used to set up an 'Engine' during registration.

        // Define a custom operator '@' with precedence of 160
        // (i.e. between +|- and *|/).
        engine.register_custom_operator("@", 160).unwrap();
    }
}
```
