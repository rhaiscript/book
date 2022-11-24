Dynamic Values
==============

{{#include ../links.md}}

A `Dynamic` value can be _any_ type, as long as it implements `Clone`.

~~~admonish warning.small "`Send + Sync`"

Under the [`sync`] feature, all types must also be `Send + Sync`.
~~~

```rust
let x = 42;         // value is an integer

x = 123.456;        // value is now a floating-point number

x = "hello";        // value is now a string

x = x.len > 0;      // value is now a boolean

x = [x];            // value is now an array

x = #{x: x};        // value is now an object map
```


Use `type_of()` to Get Value Type
---------------------------------

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


Parse from JSON
---------------

~~~admonish warning.side "Requires `metadata`"

`parse_json` is defined in the [`LanguageCorePackage`][built-in packages], which is excluded when using a [raw `Engine`].

It also requires the [`metadata`] feature; the [`no_index`] and [`no_object`] features must _not_ be set.
~~~

Use `parse_json` to parse a JSON string into a [`Dynamic`] value.

|           JSON type           |  Rhai type   |
| :---------------------------: | :----------: |
|  `number` (no decimal point)  |    `INT`     |
| `number` (with decimal point) |   `FLOAT`    |
|           `string`            |   [string]   |
|           `boolean`           |    `bool`    |
|            `Array`            |   [array]    |
|           `Object`            | [object map] |
|            `null`             |    [`()`]    |
