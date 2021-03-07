Working With Rust Enums
=======================

{{#include ../links.md}}

Enums in Rust are typically used with _pattern matching_.

Rhai is dynamic, so although it integrates with Rust enum variants just fine (treated transparently
as [custom types]), it is impossible  (short of registering a complete API) to distinguish between
individual enum variants or to extract internal data from them.


Simulate an Enum API
--------------------

A [plugin module] is extremely handy in creating an entire API for a custom enum type.

```rust,no_run
use rhai::plugin::*;
use rhai::{Dynamic, Engine, EvalAltResult};

#[derive(Debug, Clone, Eq, PartialEq, Hash)]
enum MyEnum {
    Foo,
    Bar(i64),
    Baz(String, bool),
}

// Create a plugin module with functions constructing the 'MyEnum' variants
#[export_module]
mod MyEnumModule {
    // Constructors for 'MyEnum' variants
    pub const Foo: &MyEnum = MyEnum::Foo;
    pub fn Bar(value: i64) -> MyEnum { MyEnum::Bar(value) }
    pub fn Baz(val1: String, val2: bool) -> MyEnum { MyEnum::Baz(val1, val2) }
    // Access to fields
    #[rhai_fn(global, get = "enum_type", pure)]
    pub fn get_type(my_enum: &mut MyEnum) -> String {
        match my_enum {
            MyEnum::Foo => "Foo".to_string(),
            MyEnum::Bar(_) => "Bar".to_string(),
            MyEnum::Baz(_, _) => "Baz".to_string(),
        }
    }
    #[rhai_fn(global, get = "field_0", pure)]
    pub fn get_field_0(my_enum: &mut MyEnum) -> Dynamic {
        match my_enum {
            MyEnum::Foo => Dynamic::UNIT,
            MyEnum::Bar(x) => Dynamic::from(x),
            MyEnum::Baz(x, _) => Dynamic::from(x),
        }
    }
    #[rhai_fn(global, get = "field_1", pure)]
    pub fn get_field_1(my_enum: &mut MyEnum) -> Dynamic {
        match my_enum {
            MyEnum::Foo | MyEnum::Bar(_) => Dynamic::UNIT,
            MyEnum::Baz(_, x) => Dynamic::from(x),
        }
    }
    // Printing
    #[rhai_fn(global, name = "to_string", name = "print", name = "to_debug", name = "debug", pure)]
    pub fn to_string(my_enum: &mut MyEnum) -> String {
        format!("{:?}", my_enum)
    }
    #[rhai_fn(global, name = "+")]
    pub fn add_to_str(string: &str, my_enum: MyEnum) -> String {
        format!("{}{:?}", string, my_enum)
    }
    #[rhai_fn(global, name = "+", pure)]
    pub fn add_str(my_enum: &mut MyEnum, string: &str) -> String {
        format!("{:?}", my_enum).push_str(string)
    }
    #[rhai_fn(global, name = "+=")]
    pub fn append_to_str(s: &mut ImmutableString, my_enum: MyEnum) -> String {
        s += my_enum.to_string()
    }
    // '==' and '!=' operators
    #[rhai_fn(global, name = "==", pure)]
    pub fn eq(my_enum: &mut MyEnum, my_enum2: MyEnum) -> bool {
        my_enum == &my_enum2
    }
    #[rhai_fn(global, name = "!=", pure)]
    pub fn neq(my_enum: &mut MyEnum, my_enum2: MyEnum) -> bool {
        my_enum != &my_enum2
    }
}

let mut engine = Engine::new();

// Load the module as the module namespace "MyEnum"
engine.register_type_with_name::<MyEnum>("MyEnum")
      .register_static_module("MyEnum", exported_module!(MyEnumModule).into());
```

With this API in place, working with enums feels almost the same as in Rust:

```rust,no_run
let x = MyEnum::Foo;

let y = MyEnum::Bar(42);

let z = MyEnum::Baz("hello", true);

x == MyEnum::Foo;

y != MyEnum::Bar(0);

// Detect enum types

x.enum_type == "Foo";

y.enum_type == "Bar";

z.enum_type == "Baz";

// Extract enum fields

y.field_0 == 42;

y.field_1 == ();

z.field_0 == "hello";

z.field_1 == true;
```

Since enums are internally treated as [custom types], they are not _literals_ and cannot be
used as a match case in `switch` expressions.  This is quite a limitation because the equivalent
`match` statement is commonly used in Rust to work with enums and bind variables to
variant-internal data.

It is possible, however, to `switch` through enum variants based on their types:

```c,no_run
switch my_enum.enum_type {
  "Foo" => ...,
  "Bar" => {
    let value = foo.field_0;
    ...
  }
  "Baz" => {
    let val1 = foo.field_0;
    let val2 = foo.field_1;
    ...
  }
}
```


Use `switch` Through Arrays
---------------------------

Another way to work with Rust enums in a `switch` expression is through exposing the internal data
(or at least those that act as effective _discriminants_) of each enum variant as a variable-length
[array], usually with the name of the variant as the first item for convenience:

```rust,no_run
use rhai::Array;

engine.register_get("enum_data", |my_enum: &mut MyEnum| {
    match my_enum {
        MyEnum::Foo => vec![ "Foo".into() ] as Array,

        // Say, skip the data field because it is not
        // used as a discriminant
        MyEnum::Bar(value) => vec![ "Bar".into() ] as Array,

        // Say, all fields act as discriminants
        MyEnum::Baz(val1, val2) => vec![
            "Baz".into(), val1.clone().into(), (*val2).into()
        ] as Array
    }
});
```

Then it is a simple matter to match an enum via the `switch` expression:

```c,no_run
// Assume 'value' = 'MyEnum::Baz("hello", true)'
// 'enum_data' creates a variable-length array with 'MyEnum' data
let x = switch value.enum_data {
    ["Foo"] => 1,
    ["Bar"] => value.field_1,
    ["Baz", "hello", false] => 4,
    ["Baz", "hello", true] => 5,
    _ => 9
};

x == 5;

// Which is essentially the same as:
let x = switch [value.type, value.field_0, value.field_1] {
    ["Foo", (), ()] => 1,
    ["Bar", 42, ()] => 42,
    ["Bar", 123, ()] => 123,
            :
    ["Baz", "hello", false] => 4,
    ["Baz", "hello", true] => 5,
    _ => 9
}
```

Usually, a helper method returns an array of values that can uniquely determine
the switch case based on actual usage requirements &ndash; which means that it probably
skips fields that contain data instead of discriminants.

Then `switch` is used to very quickly match through a large number of array shapes
and jump to the appropriate case implementation.

Data fields can then be extracted from the enum independently.
