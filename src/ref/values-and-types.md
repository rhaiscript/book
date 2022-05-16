Value Types
===========

The following primitive value types are supported natively.

| Category                                                                                                     | [`type_of()`](type-of.md) | `to_string()`                   |
| ------------------------------------------------------------------------------------------------------------ | ------------------------- | ------------------------------- |
| **System integer**                                                                                           | `"i32"` or `"i64"`        | `"42"`, `"123"` etc.            |
| **Other integer number**                                                                                     | `"i32"`, `"u64"` etc.     | `"42"`, `"123"` etc.            |
| **Integer numeric [range](ranges.md)**                                                                       | `"range"`, `"range="`     | `"2..7"`, `"0..=15"` etc.       |
| **Floating-point number**                                                                                    | `"f32"` or `"f64"`        | `"123.4567"` etc.               |
| **Fixed precision decimal number**                                                                           | `"decimal"`               | `"42"`, `"123.4567"` etc.       |
| **Boolean value**                                                                                            | `"bool"`                  | `"true"` or `"false"`           |
| **Unicode character**                                                                                        | `"char"`                  | `"A"`, `"x"` etc.               |
| **Immutable Unicode [string](strings-chars.md)**                                                             | `"string"`                | `"hello"` etc.                  |
| **[`Array`](arrays.md)**                                                                                     | `"array"`                 | `"[ 1, 2, 3 ]"` etc.            |
| **Byte array &ndash; [`BLOB`](blobs.md)**                                                                    | `"blob"`                  | `"[01020304abcd]"` etc.         |
| **[Object map](object-maps.md)**                                                                             | `"map"`                   | `"#{ "a": 1, "b": true }"` etc. |
| **[Timestamp](timestamps.md)**                                                                               | `"timestamp"`             | `"<timestamp>"`                 |
| **[Function pointer](fn-ptr.md)**                                                                            | `"Fn"`                    | `"Fn(foo)"` etc.                |
| **Dynamic value** (i.e. can be anything)                                                                     | _the actual type_         | _actual value_                  |
| **Shared value** (a reference-counted, shared dynamic value, created via [automatic currying](fn-closure.md) | _the actual type_         | _actual value_                  |
| **Nothing/void/nil/null/Unit** (or whatever it is called)                                                    | `"()"`                    | `""` _(empty string)_           |


```admonish warning.small "All types are distinct"

All types are treated strictly distinct by Rhai, meaning that `i32` and `i64` and `u32` are
completely different. They cannot even be added together.

This is very similar to Rust.
```

```admonish info.small "Strings"

[Strings](strings-chars.md) in Rhai are _immutable_, meaning that they can be shared but not modified.

Any modification done to a Rhai string causes the [string](strings-chars.md) to be cloned and
the modifications made to the copy.
```

```admonish tip.small "Tip: Convert to string"

The `to_string` function converts a standard type into a [string](strings-chars.md) for display purposes.

The `to_debug` function converts a standard type into a [string](strings-chars.md) in debug format.
```
