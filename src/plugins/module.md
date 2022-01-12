Export a Rust Module to Rhai
============================

{{#include ../links.md}}


Import Prelude
--------------

When using the plugins system, the entire `rhai::plugin` module must be imported as a prelude
because code generated will need these imports.

```rust,no_run
use rhai::plugin::*;
```


`#[export_module]`
------------------

When applied to a Rust module, the `#[export_module]` attribute generates the necessary
code and metadata to allow Rhai access to its public (i.e. marked `pub`) functions, constants
and sub-modules.

This code is exactly what would need to be written by hand to achieve the same goal,
and is custom fit to each exported item.

All `pub` functions become registered functions, all `pub` constants become [module] constant variables,
and all sub-modules become Rhai sub-modules.

This Rust module can then be registered into an [`Engine`] as a normal [module].
This is done via the `exported_module!` macro.

The macro `combine_with_exported_module!` can be used to _combine_ all the functions
and variables into an existing [module], _flattening_ the namespace &ndash; i.e. all sub-modules
are eliminated and their contents promoted to the top level.  This is typical for
developing [custom packages].

```rust,no_run
use rhai::plugin::*;        // a "prelude" import for macros

#[export_module]
mod my_module {
    // This constant will be registered as the constant variable 'MY_NUMBER'.
    // Ignored when registered as a global module.
    pub const MY_NUMBER: i64 = 42;

    // This function will be registered as 'greet'
    // but is only available with the 'greetings' feature.
    #[cfg(feature = "greetings")]
    pub fn greet(name: &str) -> String {
        format!("hello, {}!", name)
    }
    /// This function will be registered as 'get_num'.
    ///
    /// If this is a Rust doc-comment, then it is included in the metadata.
    pub fn get_num() -> i64 {
        mystic_number()
    }
    // This function will be registered as 'increment'.
    // It will also be exposed to the global namespace since 'global' is set.
    #[rhai_fn(global)]
    pub fn increment(num: &mut i64) {
        *num += 1;
    }
    // This function is not 'pub', so NOT registered.
    fn mystic_number() -> i64 {
        42
    }

    // Sub-modules are ignored when the module is registered globally.
    pub mod my_sub_module {
        // This function is ignored when registered globally.
        // Otherwise it is a valid registered function under a sub-module.
        pub fn get_info() -> String {
            "hello".to_string()
        }
    }

    // Sub-modules are commonly used to put feature gates on a group of
    // functions because feature gates cannot be put on function definitions.
    // This is currently a limitation of the plugin procedural macros.
    #[cfg(feature = "advanced_functions")]
    pub mod advanced {
        // This function is ignored when registered globally.
        // Otherwise it is a valid registered function under a sub-module
        // which only exists when the 'advanced_functions' feature is used.
        pub fn advanced_calc(input: i64) -> i64 {
            input * 2
        }
    }
}
```

### Doc-comments

If the [`metadata`] feature is active, doc-comments (i.e. comments starting with `///` or wrapped
with `/**` ... `*/`) on plugin functions are extracted into the functions' [_metadata_][functions metadata].


### Use `Engine::register_global_module`

The simplest way to register this into an [`Engine`] is to first use the `exported_module!` macro
to turn it into a normal Rhai [module], then use the `Engine::register_global_module` method on it:

```rust,no_run
fn main() {
    let mut engine = Engine::new();

    // The macro call creates a Rhai module from the plugin module.
    let module = exported_module!(my_module);

    // A module can simply be registered into the global namespace.
    engine.register_global_module(module.into());
}
```

The functions contained within the module definition (i.e. `greet`, `get_num` and `increment`)
are automatically registered into the [`Engine`] when `Engine::register_global_module` is called.

```rust,no_run
let x = greet("world");
x == "hello, world!";

let x = greet(get_num().to_string());
x == "hello, 42!";

let x = get_num();
x == 42;

increment(x);
x == 43;
```

Notice that, when using a [module] as a [package], only functions registered at the _top level_
can be accessed.

Variables as well as sub-modules are **ignored**.

### Use `Engine::register_static_module`

Another simple way to register this into an [`Engine`] is, again, to use the `exported_module!` macro
to turn it into a normal Rhai [module], then use `Engine::register_static_module` on it.

```rust,no_run
fn main() {
    let mut engine = Engine::new();

    // The macro call creates a Rhai module from the plugin module.
    let module = exported_module!(my_module);

    // A module can simply be registered as a static module namespace.
    engine.register_static_module("service", module.into());
}
```

The functions contained within the module definition (i.e. `greet`, `get_num` and `increment`),
plus the constant `MY_NUMBER`, are automatically registered under the module namespace `service`:

```rust,no_run
let x = service::greet("world");
x == "hello, world!";

service::MY_NUMBER == 42;

let x = service::greet(service::get_num().to_string());
x == "hello, 42!";

let x = service::get_num();
x == 42;

service::increment(x);
x == 43;
```

All functions (usually _methods_) defined in the module and marked with `#[rhai_fn(global)]`,
as well as all [type iterators], are automatically exposed to the _global_ namespace, so
[iteration][`for`], [getters/setters] and [indexers] for [custom types]
can work as expected.

In fact, the default for all [getters/setters] and [indexers] defined in a plugin module
is `#[rhai_fn(global)]` unless specifically overridden by `#[rhai_fn(internal)]`.

Therefore, in the example above, the `increment` method (defined with `#[rhai_fn(global)]`)
works fine when called in method-call style:

```rust,no_run
let x = 42;
x.increment();
x == 43;
```

### Use Dynamically

Using this directly as a dynamically-loadable Rhai [module] is almost the same, except that a
[module resolver] must be used to serve the module, and the module is loaded via `import` statements.

See the [module] section for more information.

### Combine into Custom Package

Finally the plugin module can also be used to develop a [custom package], using
`combine_with_exported_module!` which automatically _flattens_ the module namespace so that all
functions in sub-modules are promoted to the top level [namespace][function namespace],
all sub-modules are eliminated, and all variables are ignored.

Due to this _flattening_, sub-modules are used conveniently as a _grouping_ mechanism,
especially to put _feature gates_ or _compile-time gates_ (i.e. `#[cfg(...)]`) on a
large collection of functions without having to duplicate the gates onto individual functions.

```rust,no_run
#[export_module]
mod my_module {
    // Always available
    pub fn func0() {}

    // The following functions are only available under 'foo'.
    // Use a sub-module for convenience, since all functions underneath
    // will be flattened into the namespace.
    #[cfg(feature = "foo")]
    pub mod group_foo {
        pub fn func1() {}
        pub fn func2() {}
        pub fn func3() {}
    }

    // The following functions are only available under 'bar'
    #[cfg(feature = "bar")]
    pub mod group_bar {
        pub fn func4() {}
        pub fn func5() {}
        pub fn func6() {}
    }
}

// The above is equivalent to:
#[export_module]
mod my_module_alternate {
    pub fn func0() {}

    #[cfg(feature = "foo")]
    pub fn func1() {}
    #[cfg(feature = "foo")]
    pub fn func2() {}
    #[cfg(feature = "foo")]
    pub fn func3() {}

    #[cfg(feature = "bar")]
    pub fn func4() {}
    #[cfg(feature = "bar")]
    pub fn func5() {}
    #[cfg(feature = "bar")]
    pub fn func6() {}
}

// Registered functions:
//   func0 - always available
//   func1, func2, func3 - available under 'foo'
//   func4, func5, func6 - available under 'bar'
//   func0, func1, func2, func3, func4, func5, func6 - available under 'foo' and 'bar'
combine_with_exported_module!(module, "my_module_ID", my_module);
```


Functions Overloading and Operators
----------------------------------

Operators and overloaded functions can be specified via applying the `#[rhai_fn(name = "...")]`
attribute to individual functions.

The text string given as the `name` parameter to `#[rhai_fn]` is used to register the function with
the [`Engine`], disregarding the actual name of the function.

With `#[rhai_fn(name = "...")]`, multiple functions may be registered under the same name in Rhai,
so long as they have different parameters.

Operators (which require function names that are not valid for Rust) can also be registered this way.

Registering the same function name with the same parameter types will cause a parsing error.

```rust,no_run
use rhai::plugin::*;        // a "prelude" import for macros

#[export_module]
mod my_module {
    // This is the '+' operator for 'TestStruct'.
    #[rhai_fn(name = "+")]
    pub fn add(obj: &mut TestStruct, value: i64) {
        obj.prop += value;
    }
    // This function is 'calc (i64)'.
    pub fn calc(num: i64) -> i64 {
        ...
    }
    // This function is 'calc (i64, bool)'.
    #[rhai_fn(name = "calc")]
    pub fn calc_with_option(num: i64, option: bool) -> i64 {
        ...
    }
}
```


Getters, Setters and Indexers
-----------------------------

Functions can be marked as [getters/setters] and [indexers] for [custom types]
via the `#[rhai_fn]` attribute, which is applied on a function level.

```rust,no_run
use rhai::plugin::*;        // a "prelude" import for macros

#[export_module]
mod my_module {
    // This is a normal function 'greet'.
    pub fn greet(name: &str) -> String {
        format!("hello, {}!", name)
    }
    // This is a getter for 'TestStruct::prop'.
    #[rhai_fn(get = "prop", pure)]
    pub fn get_prop(obj: &mut TestStruct) -> i64 {
        obj.prop
    }
    // This is a setter for 'TestStruct::prop'.
    #[rhai_fn(set = "prop")]
    pub fn set_prop(obj: &mut TestStruct, value: i64) {
        obj.prop = value;
    }
    // This is an index getter for 'TestStruct'.
    #[rhai_fn(index_get)]
    pub fn get_index(obj: &mut TestStruct, index: i64) -> bool {
        obj.list[index]
    }
    // This is an index setter for 'TestStruct'.
    #[rhai_fn(index_set)]
    pub fn get_index(obj: &mut TestStruct, index: i64, state: bool) {
        obj.list[index] = state;
    }
}
```


Multiple Registrations
----------------------

Parameters to the `#[rhai_fn(...)]` attribute can be applied multiple times.

This is especially useful for the `name = "..."`, `get = "..."` and `set = "..."` parameters
to give multiple alternative names to the same function.

```rust,no_run
use rhai::plugin::*;        // a "prelude" import for macros

#[export_module]
mod my_module {
    // This function can be called in five ways
    #[rhai_fn(name = "get_prop_value", name = "prop", name = "+", set = "prop", index_get)]
    pub fn prop_function(obj: &mut TestStruct, index: i64) -> i64 {
        obj.prop[index]
    }
}
```

The above function can be called in five ways:

| Parameter for `#[rhai_fn(...)]` |           Type            | Call style                                    |
| ------------------------------- | :-----------------------: | --------------------------------------------- |
| `name = "get_prop_value"`       |      method function      | `get_prop_value(x, 0)`, `x.get_prop_value(0)` |
| `name = "prop"`                 |      method function      | `prop(x, 0)`, `x.prop(0)`                     |
| `name = "+"`                    |        [operator]         | `x + 42`                                      |
| `set = "prop"`                  | [setter][getters/setters] | `x.prop = 42`                                 |
| `index_get`                     |  [index getter][indexer]  | `x[0]`                                        |


Pure Functions
--------------

Apply the `#[rhai_fn(pure)]` attribute on a method function (i.e. one taking a `&mut` first parameter)
to mark it as  _pure_.

Pure functions _MUST NOT_ modify the value of the `&mut` parameter.

Therefore, pure functions can be passed a [constant] value as the first `&mut` parameter.

Non-pure functions, when passed a [constant] value as the first `&mut` parameter, will raise an
`EvalAltResult::ErrorAssignmentToConstant` error.

```rust,no_run
use rhai::plugin::*;        // a "prelude" import for macros

#[export_module]
mod my_module {
    // This function can be passed a constant
    #[rhai_fn(name = "add1", pure)]
    pub fn add_scaled(array: &mut rhai::Array, x: i64) -> i64 {
        array.iter().map(|v| v.as_int().unwrap()).fold(0, |(r, v)| r += v * x)
    }
    // This function CANNOT be passed a constant
    #[rhai_fn(name = "add2")]
    pub fn add_scaled2(array: &mut rhai::Array, x: i64) -> i64 {
        array.iter().map(|v| v.as_int().unwrap()).fold(0, |(r, v)| r += v * x)
    }
    // This getter can be applied to a constant
    #[rhai_fn(get = "first1", pure)]
    pub fn get_first(array: &mut rhai::Array) -> i64 {
        array[0]
    }
    // This getter CANNOT be applied to a constant
    #[rhai_fn(get = "first2")]
    pub fn get_first2(array: &mut rhai::Array) -> i64 {
        array[0]
    }
    // The following is a syntax error because a setter is SUPPOSED to
    // mutate the object.  Therefore the 'pure' attribute cannot be used.
    #[rhai_fn(get = "values", pure)]
    pub fn set_values(array: &mut rhai::Array, value: i64) {
        // ...
    }
}
```

When applied to a Rhai script:

```rust,no_run
// Constant
const VECTOR = [1, 2, 3, 4, 5, 6, 7];

let r = VECTOR.add1(2);     // ok!

let r = VECTOR.add2(2);     // runtime error: constant modified

let r = VECTOR.first1;      // ok!

let r = VECTOR.first2;      // runtime error: constant modified
```


Fallible Functions
------------------

To register [fallible functions] (i.e. functions that may return errors), apply the
`#[rhai_fn(return_raw)]` attribute on functions that return `Result<T, Box<EvalAltResult>>`
where `T` is any clonable type.

A syntax error is generated if the function with `#[rhai_fn(return_raw)]` does not
have the appropriate return type.

```rust,no_run
use rhai::plugin::*;        // a "prelude" import for macros

#[export_module]
mod my_module {
    /// This overloads the '/' operator for i64.
    #[rhai_fn(name = "/", return_raw)]
    pub fn double_and_divide(x: i64, y: i64) -> Result<i64, Box<EvalAltResult>> {
        if y == 0 {
            Err("Division by zero!".into())
        } else {
            Ok((x * 2) / y)
        }
    }
}
```


`NativeCallContext` Parameter
----------------------------

The _first_ parameter of a function can also be [`NativeCallContext`], which is treated
specially by the plugins system.


`#[export_module]` Parameters
----------------------------

Parameters can be applied to the `#[export_module]` attribute to override its default behavior.

| Parameter               | Description                                                                                                                    |
| ----------------------- | ------------------------------------------------------------------------------------------------------------------------------ |
| _none_                  | exports only public (i.e. `pub`) functions                                                                                     |
| `export_all`            | exports all functions (including private, non-`pub` functions); use `#[rhai_fn(skip)]` on individual functions to avoid export |
| `export_prefix = "..."` | exports functions (including private, non-`pub` functions) with names starting with a specific prefix                          |


Inner Attributes
----------------

Inner attributes can be applied to the inner items of a module to tweak the export process.

`#[rhai_fn]` is applied to functions, while `#[rhai_mod]` is applied to sub-modules.

Parameters should be set on inner attributes to specify the desired behavior.

| Attribute Parameter | Use with                       | Apply to                                           | Description                                                                   |
| ------------------- | ------------------------------ | -------------------------------------------------- | ----------------------------------------------------------------------------- |
| `skip`              | `#[rhai_fn]`<br/>`#[rhai_mod]` | function or sub-module                             | do not export this function/[sub-module][module]                              |
| `global`            | `#[rhai_fn]`                   | function                                           | expose this function to the global [namespace][function namespace]            |
| `internal`          | `#[rhai_fn]`                   | function                                           | keep this function within the internal module [namespace][function namespace] |
| `name = "..."`      | `#[rhai_fn]`<br/>`#[rhai_mod]` | function or sub-module                             | registers function/[sub-module][module] under the specified name              |
| `get = "..."`       | `#[rhai_fn]`                   | `pub fn (&mut Type) -> ValueType`                  | registers a property [getter][getters/setters] for the named property         |
| `set = "..."`       | `#[rhai_fn]`                   | `pub fn (&mut Type, ValueType)`                    | registers a property [setter][getters/setters] for the named property         |
| `index_get`         | `#[rhai_fn]`                   | `pub fn (&mut Type, IndexType) -> ValueType`       | registers an [index getter][indexer]                                          |
| `index_set`         | `#[rhai_fn]`                   | `pub fn (&mut Type, IndexType, ValueType)`         | registers an [index setter][indexer]                                          |
| `return_raw`        | `#[rhai_fn]`                   | `pub fn (...) -> Result<Type, Box<EvalAltResult>>` | marks this as a [fallible function]                                           |
| `pure`              | `#[rhai_fn]`                   | `pub fn (&mut Type, ...) -> ...`                   | marks this as a _pure_ function                                               |
