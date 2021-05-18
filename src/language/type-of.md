`type_of()`
===========

{{#include ../links.md}}

The `type_of` function detects the actual type of a value.

This is useful because all variables are [`Dynamic`] in nature.

```js
// Use 'type_of()' to get the actual types of values
type_of('c') == "char";
type_of(42) == "i64";

let x = 123;
x.type_of() == "i64";       // method-call style is also OK
type_of(x) == "i64";

x = 99.999;
type_of(x) == "f64";

x = "hello";
if type_of(x) == "string" {
    do_something_first_with_string(x);
}

switch type_of(x) {
    "string" => do_something_with_string(x),
    "char" => do_something_with_char(x),
    "i64" => do_something_with_int(x),
    "f64" => do_something_with_float(x),
    "bool" => do_something_with_bool(x),
    _ => throw `I cannot work with ${type_of(x)}!!!`
}
```


Custom Types
------------

`type_of()` a [custom type] returns:

* the registered name, if registered via `Engine::register_type_with_name`

* the full Rust type name, if registered via `Engine::register_type`

```rust , no_run
struct TestStruct1;
struct TestStruct2;

engine
    // type_of(struct1) == "crate::path::to::module::TestStruct1"
    .register_type::<TestStruct1>()
    // type_of(struct2) == "MyStruct"
    .register_type_with_name::<TestStruct2>("MyStruct");
```
