Dynamic Values
==============

{{#include ../links.md}}

A `Dynamic` value can be _any_ type, as long as it implements `Clone`.

~~~admonish warning.small "`Send + Sync`"

Under the [`sync`] feature, all types must also be `Send + Sync`.
~~~


Use `type_of()` to Get Value Type
--------------------------------

Because [`type_of()`] a `Dynamic` value returns the type of the actual value,
it is usually used to perform type-specific actions based on the actual value's type.

```js
let mystery = get_some_dynamic_value();

switch type_of(mystery) {
    "()" => print("Hey, I got the unit () here!"),
    "i64" => print("Hey, I got an integer here!"),
    "f64" => print("Hey, I got a float here!"),
    "decimal" => print("Hey, I got a decimal here!"),
    "range" => print("Hey, I got an exclusive range here!"),
    "range=" => print("Hey, I got an inclusive range here!"),
    "string" => print("Hey, I got a string here!"),
    "bool" => print("Hey, I got a boolean here!"),
    "array" => print("Hey, I got an array here!"),
    "blob" => print("Hey, I got a BLOB here!"),
    "map" => print("Hey, I got an object map here!"),
    "Fn" => print("Hey, I got a function pointer here!"),
    "timestamp" => print("Hey, I got a time-stamp here!"),
    "TestStruct" => print("Hey, I got the TestStruct custom type here!"),
    _ => print(`I don't know what this is: ${type_of(mystery)}`)
}
```


Type Checking and Casting
------------------------

~~~admonish tip.side.wide "Tip: `Dynamic::try_cast`"

The `try_cast` method does not panic but returns `None` upon failure.
~~~

A `Dynamic` value's actual type can be checked via `Dynamic::is`.

The `cast` method then converts the value into a specific, known type.

Use `clone_cast` to clone a reference to `Dynamic`.

```rust
let list: Array = engine.eval("...")?;      // return type is 'Array'
let item = list[0].clone();                 // an element in an 'Array' is 'Dynamic'

item.is::<i64>() == true;                   // 'is' returns whether a 'Dynamic' value is of a particular type

let value = item.cast::<i64>();             // if the element is 'i64', this succeeds; otherwise it panics
let value: i64 = item.cast();               // type can also be inferred

let value = item.try_cast::<i64>()?;        // 'try_cast' does not panic when the cast fails, but returns 'None'

let value = list[0].clone_cast::<i64>();    // use 'clone_cast' on '&Dynamic'
let value: i64 = list[0].clone_cast();
```


Type Name and Matching Types
----------------------------

The `type_name` method gets the name of the actual type as a static string slice,
which can be `match`-ed against.

This is a very simple and direct way to act on a `Dynamic` value based on the actual type of
the data value.

```rust
let list: Array = engine.eval("...")?;      // return type is 'Array'
let item = list[0];                         // an element in an 'Array' is 'Dynamic'

match item.type_name() {                    // 'type_name' returns the name of the actual Rust type
    "()" => ...
    "i64" => ...
    "f64" => ...
    "rust_decimal::Decimal" => ...
    "core::ops::range::Range<i64>" => ...
    "core::ops::range::RangeInclusive<i64>" => ...
    "alloc::string::String" => ...
    "bool" => ...
    "char" => ...
    "rhai::FnPtr" => ...
    "std::time::Instant" => ...
    "crate::path::to::module::TestStruct" => ...
        :
}
```

```admonish warning.small "Always full path name"

`type_name` always returns the _full_ Rust path name of the type, even when the type
has been registered with a friendly name via `Engine::register_type_with_name`.

This behavior is different from that of the [`type_of`][`type_of()`] function in Rhai.
```


Methods and Traits
------------------

The following methods are available when working with `Dynamic`:

| Method          |     Not available under     |        Return type        | Description                                                                                                                                         |
| --------------- | :-------------------------: | :-----------------------: | --------------------------------------------------------------------------------------------------------------------------------------------------- |
| `type_name`     |                             |          `&str`           | name of the value's type                                                                                                                            |
| `into_shared`   |       [`no_closure`]        |         `Dynamic`         | turn the value into a _shared_ value                                                                                                                |
| `flatten_clone` |                             |         `Dynamic`         | clone the value (a _shared_ value, if any, is cloned into a separate copy)                                                                          |
| `flatten`       |                             |         `Dynamic`         | clone the value into a separate copy if it is _shared_ and there are multiple outstanding references, otherwise _shared_ values are turned unshared |
| `read_lock<T>`  | [`no_closure`] (pass thru') | `Option<` _guard to_ `T>` | lock the value for reading                                                                                                                          |
| `write_lock<T>` | [`no_closure`] (pass thru') | `Option<` _guard to_ `T>` | lock the value exclusively for writing                                                                                                              |

### Constructor instance methods

| Method           | Not available under |                         Value type                          |         Data type         |
| ---------------- | :-----------------: | :---------------------------------------------------------: | :-----------------------: |
| `from_bool`      |                     |                           `bool`                            |          `bool`           |
| `from_int`       |                     |                            `INT`                            |      integer number       |
| `from_float`     |    [`no_float`]     |                           `FLOAT`                           |   floating-point number   |
| `from_decimal`   |   non-[`decimal`]   |                  [`Decimal`][rust_decimal]                  | [`Decimal`][rust_decimal] |
| `from_str`       |                     |                           `&str`                            |         [string]          |
| `from_char`      |                     |                           `char`                            |        [character]        |
| `from_array`     |    [`no_index`]     |                          `Vec<T>`                           |          [array]          |
| `from_blob`      |    [`no_index`]     |                          `Vec<u8>`                          |          [BLOB]           |
| `from_map`       |    [`no_object`]    |                            `Map`                            |       [object map]        |
| `from_timestamp` |     [`no_std`]      | `std::time::Instant` ([`instant::Instant`] if [WASM] build) |        [timestamp]        |
| `from<T>`        |                     |                             `T`                             |       [custom type]       |

### Detection methods

| Method         | Not available under | Return type | Description                                                            |
| -------------- | :-----------------: | :---------: | ---------------------------------------------------------------------- |
| `is<T>`        |                     |   `bool`    | is the value of type `T`?                                              |
| `is_variant`   |                     |   `bool`    | is the value a trait object (i.e. not one of Rhai's [standard types])? |
| `is_read_only` |                     |   `bool`    | is the value [constant]? A [constant] value should not be modified.    |
| `is_shared`    |   [`no_closure`]    |   `bool`    | is the value _shared_ via a [closure]?                                 |
| `is_locked`    |   [`no_closure`]    |   `bool`    | is the value _shared_ and locked (i.e. currently being read)?          |

### Casting methods

The following methods cast a `Dynamic` into a specific type:

| Method                  | Not available under |     Return type (error is the actual data type)      |
| ----------------------- | :-----------------: | :--------------------------------------------------: |
| `cast<T>`               |                     |               `T` (panics on failure)                |
| `try_cast<T>`           |                     |                     `Option<T>`                      |
| `clone_cast<T>`         |                     |        cloned copy of `T` (panics on failure)        |
| `as_unit`               |                     |                  `Result<(), &str>`                  |
| `as_int`                |                     |                 `Result<INT, &str>`                  |
| `as_float`              |    [`no_float`]     |                `Result<FLOAT, &str>`                 |
| `as_decimal`            |   non-[`decimal`]   |       [`Result<Decimal, &str>`][rust_decimal]        |
| `as_bool`               |                     |                 `Result<bool, &str>`                 |
| `as_char`               |                     |                 `Result<char, &str>`                 |
| `into_string`           |                     |                `Result<String, &str>`                |
| `into_immutable_string` |                     | [`Result<ImmutableString, &str>`][`ImmutableString`] |
| `into_array`            |    [`no_index`]     |            [`Result<Array, &str>`][array]            |
| `into_blob`             |    [`no_index`]     |             [`Result<Blob, &str>`][BLOB]             |
| `into_typed_array<T>`   |    [`no_index`]     |                `Result<Vec<T>, &str>`                |

### Constructor traits

The following constructor traits are implemented for `Dynamic` where `T: Clone`:

| Trait                                                                          |      Not available under       |         Data type         |
| ------------------------------------------------------------------------------ | :----------------------------: | :-----------------------: |
| `From<()>`                                                                     |                                |           `()`            |
| `From<INT>`                                                                    |                                |      integer number       |
| `From<FLOAT>`                                                                  |          [`no_float`]          |   floating-point number   |
| `From<Decimal>`                                                                |        non-[`decimal`]         | [`Decimal`][rust_decimal] |
| `From<bool>`                                                                   |                                |          `bool`           |
| `From<S: Into<ImmutableString>>`<br/>e.g. `From<String>`, `From<&str>`         |                                |    [`ImmutableString`]    |
| `From<char>`                                                                   |                                |        [character]        |
| `From<Vec<T>>`                                                                 |          [`no_index`]          |          [array]          |
| `From<&[T]>`                                                                   |          [`no_index`]          |          [array]          |
| `From<BTreeMap<K: Into<SmartString>, T>>`<br/>e.g. `From<BTreeMap<String, T>>` |         [`no_object`]          |       [object map]        |
| `From<BTreeSet<K: Into<SmartString>>>`<br/>e.g. `From<BTreeSet<String>>`       |         [`no_object`]          |       [object map]        |
| `From<HashMap<K: Into<SmartString>, T>>`<br/>e.g. `From<HashMap<String, T>>`   |  [`no_object`] or [`no_std`]   |       [object map]        |
| `From<HashSet<K: Into<SmartString>>>`<br/>e.g. `From<HashSet<String>>`         |  [`no_object`] or [`no_std`]   |       [object map]        |
| `From<FnPtr>`                                                                  |                                |    [function pointer]     |
| `From<Instant>`                                                                |           [`no_std`]           |        [timestamp]        |
| `From<Rc<RefCell<Dynamic>>>`                                                   |   [`sync`] or [`no_closure`]   |         `Dynamic`         |
| `From<Arc<RwLock<Dynamic>>>` ([`sync`])                                        | non-[`sync`] or [`no_closure`] |         `Dynamic`         |
| `FromIterator<X: IntoIterator<Item=T>>`                                        |          [`no_index`]          |          [array]          |
