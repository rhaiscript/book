Dynamic Values
==============

{{#include ../links.md}}

A `Dynamic` value can be _any_ type. However, under [`sync`], all types must be `Send + Sync`.


Use `type_of()` to Get Value Type
--------------------------------

Because [`type_of()`] a `Dynamic` value returns the type of the actual value,
it is usually used to perform type-specific actions based on the actual value's type.

```c
let mystery = get_some_dynamic_value();

switch type_of(mystery) {
    "i64" => print("Hey, I got an integer here!"),
    "f64" => print("Hey, I got a float here!"),
    "string" => print("Hey, I got a string here!"),
    "bool" => print("Hey, I got a boolean here!"),
    "array" => print("Hey, I got an array here!"),
    "map" => print("Hey, I got an object map here!"),
    "Fn" => print("Hey, I got a function pointer here!"),
    "TestStruct" => print("Hey, I got the TestStruct custom type here!"),
    _ => print("I don't know what this is: " + type_of(mystery))
}
```


Functions Returning `Dynamic`
----------------------------

In Rust, sometimes a `Dynamic` forms part of a returned value &ndash; a good example is an [array]
which contains `Dynamic` elements, or an [object map] which contains `Dynamic` property values.

To get the _real_ values, the actual value types _must_ be known in advance.
There is no easy way for Rust to decide, at run-time, what type the `Dynamic` value is
(short of using the `type_name` function and match against the name).


Type Checking and Casting
------------------------

A `Dynamic` value's actual type can be checked via the `is` method.

The `cast` method then converts the value into a specific, known type.

Alternatively, use the `try_cast` method which does not panic but returns `None` when the cast fails.

```rust
let list: Array = engine.eval("...")?;      // return type is 'Array'
let item = list[0];                         // an element in an 'Array' is 'Dynamic'

item.is::<i64>() == true;                   // 'is' returns whether a 'Dynamic' value is of a particular type

let value = item.cast::<i64>();             // if the element is 'i64', this succeeds; otherwise it panics
let value: i64 = item.cast();               // type can also be inferred

let value = item.try_cast::<i64>()?;        // 'try_cast' does not panic when the cast fails, but returns 'None'
```

Type Name
---------

The `type_name` method gets the name of the actual type as a static string slice,
which can be `match`-ed against.

```rust
let list: Array = engine.eval("...")?;      // return type is 'Array'
let item = list[0];                         // an element in an 'Array' is 'Dynamic'

match item.type_name() {                    // 'type_name' returns the name of the actual Rust type
    "i64" => ...
    "alloc::string::String" => ...
    "bool" => ...
    "crate::path::to::module::TestStruct" => ...
}
```

**Note:** `type_name` always returns the _full_ Rust path name of the type, even when the type
has been registered with a friendly name via `Engine::register_type_with_name`.  This behavior
is different from that of the [`type_of`][`type_of()`] function in Rhai.


Methods and Traits
------------------

The following methods are available when working with `Dynamic`:

| Method                        |        Return type        | Description                                                                                                                                         |
| ----------------------------- | :-----------------------: | --------------------------------------------------------------------------------------------------------------------------------------------------- |
| `from<T>` _(instance method)_ |         `Dynamic`         | create a `Dynamic` from any value that implements `Clone`                                                                                           |
| `type_name`                   |          `&str`           | name of the value's type                                                                                                                            |
| `into_shared`                 |         `Dynamic`         | turn the value into a _shared_ value (not available under [`no_closure`])                                                                           |
| `flatten_clone`               |         `Dynamic`         | clone the value (a _shared_ value, if any, is cloned into a separate copy)                                                                          |
| `flatten`                     |         `Dynamic`         | clone the value into a separate copy if it is _shared_ and there are multiple outstanding references, otherwise _shared_ values are turned unshared |
| `read_lock<T>`                | `Option<` _guard to_ `T>` | lock the value for reading (no action under [`no_closure`]                                                                                          |
| `write_lock<T>`               | `Option<` _guard to_ `T>` | lock the value exclusively for writing (no action under [`no_closure`]                                                                              |

### Detection methods

| Method         | Return type | Description                                                                                       |
| -------------- | :---------: | ------------------------------------------------------------------------------------------------- |
| `is<T>`        |   `bool`    | is the value of type `T`?                                                                         |
| `is_variant`   |   `bool`    | is the value a trait object (i.e. not one of Rhai's [standard types])?                            |
| `is_shared`    |   `bool`    | is the value _shared_ via a [closure]? Always `false` under [`no_closure`].                       |
| `is_read_only` |   `bool`    | is the value [constant]? A [constant] value should not be modified.                               |
| `is_locked`    |   `bool`    | is the value _shared_ and locked (i.e. currently being read)? Always `false` under [`no_closure`] |

### Casting methods

The following methods cast a `Dynamic` into a specific type:

| Method                                        |                        Return type                         |
| --------------------------------------------- | :--------------------------------------------------------: |
| `cast<T>`                                     |                  `T` (panics on failure)                   |
| `try_cast<T>`                                 |                        `Option<T>`                         |
| `as_int`                                      | `Result<i64, &str>` (`Result<i32, &str>` if [`only_i32`])  |
| `as_float` (not available under [`no_float`]) | `Result<f64, &str>` (`Result<f32, &str>` if [`f32_float`]) |
| `as_decimal` (requires [`decimal`])           |                  `Result<Decimal, &str>`                   |
| `as_bool`                                     |                    `Result<bool, &str>`                    |
| `as_char`                                     |                    `Result<char, &str>`                    |
| `as_str`                                      |                    `Result<&str, &str>`                    |
| `take_string`                                 |                   `Result<String, &str>`                   |
| `take_immutable_string`                       |              `Result<ImmutableString, &str>`               |

### Constructor traits

The following constructor traits are implemented for `Dynamic`:

| Trait                                                                                                                |           Data type            |
| -------------------------------------------------------------------------------------------------------------------- | :----------------------------: |
| `From<i64>` (`From<i32>` if [`only_i32`])                                                                            | `i64` (`i32` if [`only_i32`])  |
| `From<f64>` (`From<f32>` if [`f32_float`], not available under [`no_float`])                                         | `f64` (`f32` if [`f32_float`]) |
| `From<Decimal>` (requires [`decimal`])                                                                               |   [`Decimal`][rust_decimal]    |
| `From<bool>`                                                                                                         |             `bool`             |
| `From<S: Into<ImmutableString>>`<br/>e.g. `From<String>`, `From<&str>`                                               |      [`ImmutableString`]       |
| `From<char>`                                                                                                         |             `char`             |
| `From<Vec<T>>` (not available under [`no_index`])                                                                    |            [array]             |
| `From<&[T]>` (not available under [`no_index`])                                                                      |            [array]             |
| `From<HashMap<K: Into<ImmutableString>, T>>` (not available under [`no_object`])<br/>e.g. `From<HashMap<String, T>>` |          [object map]          |
| `From<FnPtr>`                                                                                                        |       [function pointer]       |
| `From<Instant>` (not available under [`no_std`])                                                                     |          [timestamp]           |
