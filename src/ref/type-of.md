`type_of()`
===========

The `type_of` function detects the actual type of a value.

This is useful because all [variables](variables.md) are dynamic in nature.

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
