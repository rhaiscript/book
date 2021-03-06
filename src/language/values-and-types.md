Values and Types
===============

{{#include ../links.md}}

The following primitive types are supported natively:

| Category                                                                                                                         | Equivalent Rust types                                                                                 | [`type_of()`]         | `to_string()`             |
| -------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------- | --------------------- | ------------------------- |
| **Integer number**                                                                                                               | `u8`, `i8`, `u16`, `i16`, <br/>`u32`, `i32` (default for [`only_i32`]),<br/>`u64`, `i64` _(default)_  | `"i32"`, `"u64"` etc. | `"42"`, `"123"` etc.      |
| **Floating-point number** (disabled with [`no_float`])                                                                           | `f32` (default for [`f32_float`]), `f64` _(default)_                                                  | `"f32"` or `"f64"`    | `"123.4567"` etc.         |
| **Fixed precision decimal number** (requires [`decimal`])                                                                        | [`Decimal`][rust_decimal]                                                                             | `"decimal"`           | `"42"`, `"123.4567"` etc. |
| **Boolean value**                                                                                                                | `bool`                                                                                                | `"bool"`              | `"true"` or `"false"`     |
| **Unicode character**                                                                                                            | `char`                                                                                                | `"char"`              | `"A"`, `"x"` etc.         |
| **Immutable Unicode [string]**                                                                                                   | [`rhai::ImmutableString`][`ImmutableString`] (implemented as `Rc<SmartString>` or `Arc<SmartString>`) | `"string"`            | `"hello"` etc.            |
| **[`Array`]** (disabled with [`no_index`])                                                                                       | [`rhai::Array`][array]                                                                                | `"array"`             | `"[ 1, 2, 3 ]"`           |
| **[Object map]** (disabled with [`no_object`])                                                                                   | [`rhai::Map`][object map]                                                                             | `"map"`               | `"#{ "a": 1, "b": 2 }"`   |
| **[Timestamp]** (disabled with [`no_std`])                                                                                       | `std::time::Instant` ([`instant::Instant`] if [WASM] build)                                           | `"timestamp"`         | `"<timestamp>"`           |
| **[Function pointer]**                                                                                                           | [`rhai::FnPtr`][function pointer]                                                                     | `Fn`                  | `"Fn(foo)"`               |
| **Dynamic value** (i.e. can be anything)                                                                                         | [`rhai::Dynamic`][`Dynamic`]                                                                          | _the actual type_     | _actual value_            |
| **Shared value** (a reference-counted, shared [`Dynamic`] value, created via [automatic currying], disabled with [`no_closure`]) |                                                                                                       | _the actual type_     | _actual value_            |
| **System integer** (current configuration)                                                                                       | `rhai::INT` (`i32` or `i64`)                                                                          | `"i32"` or `"i64"`    | `"42"`, `"123"` etc.      |
| **System floating-point** (current configuration, disabled with [`no_float`])                                                    | `rhai::FLOAT` (`f32` or `f64`)                                                                        | `"f32"` or `"f64"`    | `"123.456"` etc.          |
| **Nothing/void/nil/null/Unit** (or whatever it is called)                                                                        | `()`                                                                                                  | `"()"`                | `""` _(empty string)_     |

All types are treated strictly separate by Rhai, meaning that `i32` and `i64` and `u32` are completely different -
they even cannot be added together. This is very similar to Rust.

The default integer type is `i64`. If other integer types are not needed, it is possible to exclude them and make a
smaller build with the [`only_i64`] feature.

If only 32-bit integers are needed, enabling the [`only_i32`] feature will remove support for all
integer types other than `i32`, including `i64`.
This is useful on some 32-bit targets where using 64-bit integers incur a performance penalty.

If no floating-point is needed or supported, use the [`no_float`] feature to remove it.

Some applications require fixed-precision decimal numbers, which can be enabled via the [`decimal`] feature.

[Strings] in Rhai are _immutable_, meaning that they can be shared but not modified.
Internally, the [`ImmutableString`] type is a wrapper over `Rc<String>` or `Arc<String>` (depending on [`sync`]).
Any modification done to a Rhai string causes the string to be cloned and the modifications made to the copy.

The [`to_string()`] function converts a standard type into a [string] for display purposes.

The [`to_debug()`] function converts a standard type into a [string] in debug format.
