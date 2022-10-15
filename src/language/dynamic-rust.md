Interop `Dynamic` Data with Rust
================================

{{#include ../links.md}}


Create a `Dynamic` from Rust Type
---------------------------------

| Rust type<br/>`T: Clone`,<br/>`K: Into<String>`                    |     Unavailable under     | Use API                      |
| ------------------------------------------------------------------ | :-----------------------: | ---------------------------- |
| `INT` (`i64` or `i32`)                                             |                           | `value.into()`               |
| `FLOAT` (`f64` or `f32`)                                           |       [`no_float`]        | `value.into()`               |
| [`Decimal`][rust_decimal] (requires [`decimal`])                   |                           | `value.into()`               |
| `bool`                                                             |                           | `value.into()`               |
| [`()`]                                                             |                           | `value.into()`               |
| [`String`][string], [`&str`][string], [`ImmutableString`]          |                           | `value.into()`               |
| `char`                                                             |                           | `value.into()`               |
| [`Array`][array]                                                   |       [`no_index`]        | `Dynamic::from_array(value)` |
| [`Blob`][BLOB]                                                     |       [`no_index`]        | `Dynamic::from_blob(value)`  |
| `Vec<T>`, `&[T]`, `Iterator<T>`                                    |       [`no_index`]        | `value.into()`               |
| [`Map`][object map]                                                |       [`no_object`]       | `Dynamic::from_map(value)`   |
| `HashMap<K, T>`, `HashSet<K>`,<br/>`BTreeMap<K, T>`, `BTreeSet<K>` |       [`no_object`]       | `value.into()`               |
| [`INT..INT`][range], [`INT..=INT`][range]                          |                           | `value.into()`               |
| `Rc<RwLock<T>>` or `Arc<Mutex<T>>`                                 |      [`no_closure`]       | `value.into()`               |
| [`Instant`][timestamp]                                             | [`no_time`] or [`no_std`] | `value.into()`               |
| All types (including above)                                        |                           | `Dynamic::from(value)`       |


Type Checking and Casting
-------------------------

~~~admonish tip.side "Tip: `try_cast`"

The `try_cast` method does not panic but returns `None` upon failure.
~~~

A [`Dynamic`] value's actual type can be checked via `Dynamic::is`.

The `cast` method then converts the value into a specific, known type.

Use `clone_cast` to clone a reference to [`Dynamic`].

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

This is a very simple and direct way to act on a [`Dynamic`] value based on the actual type of
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


Getting a Reference to Data
---------------------------

Use `Dynamic::read_lock` and `Dynamic::write_lock` to get an immutable/mutable reference to the data
inside a [`Dynamic`].

```rust
struct TheGreatQuestion {
    answer: i64
}

let question = TheGreatQuestion { answer: 42 };

let mut value: Dynamic = Dynamic::from(question);

let q_ref: &TheGreatQuestion =
        &*value.read_lock::<TheGreatQuestion>().unwrap();
//                       ^^^^^^^^^^^^^^^^^^^^ cast to data type

println!("answer = {}", q_ref.answer);          // prints 42

let q_mut: &mut TheGreatQuestion =
        &mut *value.write_lock::<TheGreatQuestion>().unwrap();
//                            ^^^^^^^^^^^^^^^^^^^^ cast to data type

q_mut.answer = 0;                               // mutate value

let value = value.cast::<TheGreatQuestion>();

println!("new answer = {}", value.answer);      // prints 0
```

~~~admonish question "TL;DR &ndash; Why `read_lock` and `write_lock`?"

As the naming shows, something is _locked_ in order to allow accessing the data within a [`Dynamic`],
and that something is a _shared value_ created by [capturing][automatic currying] variables from [closures].

Shared values are implemented as `Rc<RefCell<Dynamic>>` (`Arc<RwLock<Dynamic>>` under [`sync`]).

If the value is _not_ a shared value, or if running under [`no_closure`] where there is
no [capturing][automatic currying], this API de-sugars to a simple reference cast.

In other words, there is no locking and reference counting overhead for the vast majority of
non-shared values.

If the value _is_ a shared value, then it is first _locked_ and the returned _lock guard_
allows access to the underlying value in the specified type.
~~~


Methods and Traits
------------------

The following methods are available when working with [`Dynamic`]:

| Method          |     Not available under     |        Return type        | Description                                                                                                                                         |
| --------------- | :-------------------------: | :-----------------------: | --------------------------------------------------------------------------------------------------------------------------------------------------- |
| `type_name`     |                             |          `&str`           | name of the value's type                                                                                                                            |
| `into_shared`   |       [`no_closure`]        |        [`Dynamic`]        | turn the value into a _shared_ value                                                                                                                |
| `flatten_clone` |                             |        [`Dynamic`]        | clone the value (a _shared_ value, if any, is cloned into a separate copy)                                                                          |
| `flatten`       |                             |        [`Dynamic`]        | clone the value into a separate copy if it is _shared_ and there are multiple outstanding references, otherwise _shared_ values are turned unshared |
| `read_lock<T>`  | [`no_closure`] (pass thru') | `Option<` _guard to_ `T>` | lock the value for reading                                                                                                                          |
| `write_lock<T>` | [`no_closure`] (pass thru') | `Option<` _guard to_ `T>` | lock the value exclusively for writing                                                                                                              |

### Constructor instance methods

| Method           |    Not available under    |                         Value type                          |         Data type         |
| ---------------- | :-----------------------: | :---------------------------------------------------------: | :-----------------------: |
| `from_bool`      |                           |                           `bool`                            |          `bool`           |
| `from_int`       |                           |                            `INT`                            |      integer number       |
| `from_float`     |       [`no_float`]        |                           `FLOAT`                           |   floating-point number   |
| `from_decimal`   |      non-[`decimal`]      |                  [`Decimal`][rust_decimal]                  | [`Decimal`][rust_decimal] |
| `from_str`       |                           |                           `&str`                            |         [string]          |
| `from_char`      |                           |                           `char`                            |        [character]        |
| `from_array`     |       [`no_index`]        |                          `Vec<T>`                           |          [array]          |
| `from_blob`      |       [`no_index`]        |                          `Vec<u8>`                          |          [BLOB]           |
| `from_map`       |       [`no_object`]       |                            `Map`                            |       [object map]        |
| `from_timestamp` | [`no_time`] or [`no_std`] | `std::time::Instant` ([`instant::Instant`] if [WASM] build) |        [timestamp]        |
| `from<T>`        |                           |                             `T`                             |       [custom type]       |

### Detection methods

| Method         | Not available under | Return type | Description                                                            |
| -------------- | :-----------------: | :---------: | ---------------------------------------------------------------------- |
| `is<T>`        |                     |   `bool`    | is the value of type `T`?                                              |
| `is_variant`   |                     |   `bool`    | is the value a trait object (i.e. not one of Rhai's [standard types])? |
| `is_read_only` |                     |   `bool`    | is the value [constant]? A [constant] value should not be modified.    |
| `is_shared`    |   [`no_closure`]    |   `bool`    | is the value _shared_ via a [closure]?                                 |
| `is_locked`    |   [`no_closure`]    |   `bool`    | is the value _shared_ and locked (i.e. currently being read)?          |

### Casting methods

The following methods cast a [`Dynamic`] into a specific type:

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

The following constructor traits are implemented for [`Dynamic`] where `T: Clone`:

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
| `From<Instant>`                                                                |   [`no_time`] or [`no_std`]    |        [timestamp]        |
| `From<Rc<RefCell<Dynamic>>>`                                                   |   [`sync`] or [`no_closure`]   |        [`Dynamic`]        |
| `From<Arc<RwLock<Dynamic>>>` ([`sync`])                                        | non-[`sync`] or [`no_closure`] |        [`Dynamic`]        |
| `FromIterator<X: IntoIterator<Item=T>>`                                        |          [`no_index`]          |          [array]          |
