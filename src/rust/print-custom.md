Printing for Custom Types
========================

{{#include ../links.md}}

To use custom types for [`print`] and [`debug`], or convert its value into a [string],
it is necessary that the following functions be registered (assuming the custom type
is `T: Display + Debug`):

| Function    | Signature                                      | Typical implementation       | Usage                                                                |
| ----------- | ---------------------------------------------- | ---------------------------- | -------------------------------------------------------------------- |
| `to_string` | <code>\|x: &mut T\| -> String</code>           | `x.to_string()`              | converts the custom type into a [string]                             |
| `print`     | <code>\|x: &mut T\| -> String</code>           | `x.to_string()`              | converts the custom type into a [string] for the [`print`] statement |
| `to_debug`  | <code>\|x: &mut T\| -> String</code>           | `format!("{:?}", x)`         | converts the custom type into a [string] in debug format             |
| `debug`     | <code>\|x: &mut T\| -> String</code>           | `format!("{:?}", x)`         | converts the custom type into a [string] for the [`debug`] statement |
| `+`         | <code>\|s: &str, x: T\| -> String</code>       | `format!("{}{}", s, x)`      | concatenates the custom type with another [string]                   |
| `+`         | <code>\|x: &mut T, s: &str\| -> String</code>  | `x.to_string().push_str(s);` | concatenates another [string] with the custom type                   |
| `+=`        | <code>\|s: &mut ImmutableString, x: T\|</code> | `s += x.to_string()`         | appends the custom type to an existing [string]                      |
