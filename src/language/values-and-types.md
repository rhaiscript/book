Values and Types
================

{{#include ../links.md}}

The following primitive types are supported natively.

| Category                                                                                                                         | Equivalent Rust types                                                                               | [`type_of()`](type-of.md) | `to_string()`                   |
| -------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------- | ------------------------- | ------------------------------- |
| **System integer**                                                                                                               | `rhai::INT` (default `i64`, `i32` under [`only_i32`])                                               | `"i32"` or `"i64"`        | `"42"`, `"123"` etc.            |
| **Other integer number**                                                                                                         | `u8`, `i8`, `u16`, `i16`, `u32`, `i32`, `u64`, `i64`                                                | `"i32"`, `"u64"` etc.     | `"42"`, `"123"` etc.            |
| **Integer numeric [range]**                                                                                                      | `std::ops::Range<rhai::INT>`, `std::ops::RangeInclusive<rhai::INT>`                                 | `"range"`, `"range="`     | `"2..7"`, `"0..=15"` etc.       |
| **Floating-point number** (disabled with [`no_float`])                                                                           | `rhai::FLOAT` (default `f64`, `f32` under [`f32_float`])                                            | `"f32"` or `"f64"`        | `"123.4567"` etc.               |
| **Fixed precision decimal number** (requires [`decimal`])                                                                        | [`rust_decimal::Decimal`][rust_decimal]                                                             | `"decimal"`               | `"42"`, `"123.4567"` etc.       |
| **Boolean value**                                                                                                                | `bool`                                                                                              | `"bool"`                  | `"true"` or `"false"`           |
| **Unicode character**                                                                                                            | `char`                                                                                              | `"char"`                  | `"A"`, `"x"` etc.               |
| **Immutable Unicode [string]**                                                                                                   | [`rhai::ImmutableString`][`ImmutableString`] (`Rc<SmartString>`, `Arc<SmartString>` under [`sync`]) | `"string"`                | `"hello"` etc.                  |
| **[`Array`]** (disabled with [`no_index`])                                                                                       | [`rhai::Array`][array] (`Vec<Dynamic>`)                                                             | `"array"`                 | `"[ 1, 2, 3 ]"` etc.            |
| **Byte array &ndash; [`BLOB`]** (disabled with [`no_index`])                                                                     | [`rhai::Blob`][BLOB] (`Vec<u8>`)                                                                    | `"blob"`                  | `"[01020304abcd]"` etc.         |
| **[Object map]** (disabled with [`no_object`])                                                                                   | [`rhai::Map`][object map] (`BTreeMap<SmartString, Dynamic>`)                                        | `"map"`                   | `"#{ "a": 1, "b": true }"` etc. |
| **[Timestamp]** (disabled with [`no_time`] or [`no_std`])                                                                        | `std::time::Instant` ([`instant::Instant`] if [WASM] build)                                         | `"timestamp"`             | `"<timestamp>"`                 |
| **[Function pointer]**                                                                                                           | [`rhai::FnPtr`][function pointer]                                                                   | `"Fn"`                    | `"Fn(foo)"` etc.                |
| **Dynamic value** (i.e. can be anything)                                                                                         | [`rhai::Dynamic`][`Dynamic`]                                                                        | _the actual type_         | _actual value_                  |
| **Shared value** (a reference-counted, shared [`Dynamic`] value, created via [automatic currying], disabled with [`no_closure`]) |                                                                                                     | _the actual type_         | _actual value_                  |
| **Nothing/void/nil/null/Unit** (or whatever it is called)                                                                        | `()`                                                                                                | `"()"`                    | `""` _(empty string)_           |


```admonish warning.small "All types are distinct"

All types are treated strictly distinct by Rhai, meaning that `i32` and `i64` and `u32` are
completely different. They cannot even be added together.

This is very similar to Rust.
```


Default Types
-------------

The default integer type is `i64`. If other integer types are not needed, it is possible to exclude
them and make a smaller build with the [`only_i64`] feature.

If only 32-bit integers are needed, enabling the [`only_i32`] feature will remove support for all
integer types other than `i32`, including `i64`.
This is useful on some 32-bit targets where using 64-bit integers incur a performance penalty.

~~~admonish danger.small "Default integer is `i64`"

Rhai's default integer type is `i64`, which is _DIFFERENT_ from Rust's `i32`.

It is very easy to unsuspectingly set an `i32` into Rhai, which _still works_ but will incur a significant
runtime performance hit since the [`Engine`] will treat `i32` as an opaque [custom type] (unless using the
[`only_i32`] feature).
~~~

```admonish tip.small "Tip: Floating-point numbers"

If no floating-point is needed or supported, use the [`no_float`] feature to remove it.

Some applications require fixed-precision decimal numbers, which can be enabled via the [`decimal`] feature.
```

```admonish info.small "Strings"

[Strings] in Rhai are _immutable_, meaning that they can be shared but not modified.

Internally, the [`ImmutableString`] type is a wrapper over `Rc<String>` or `Arc<String>` (depending on [`sync`]).

Any modification done to a Rhai string causes the [string] to be cloned and the modifications made to the copy.
```

```admonish tip.small "Tip: Convert to string"

The [`to_string()`] function converts a standard type into a [string] for display purposes.

The [`to_debug()`] function converts a standard type into a [string] in debug format.
```
