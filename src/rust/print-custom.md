Printing for Custom Types
=========================

{{#include ../links.md}}

To use [custom types] for [`print`] and [`debug`], or convert a [custom type] into a [string],
it is necessary that the following functions be registered (assuming the [custom type]
is `T: Display + Debug`):

| Function    | Signature                            | Typical implementation | Usage                                                      |
| ----------- | ------------------------------------ | ---------------------- | ---------------------------------------------------------- |
| `to_string` | <code>\|x: &mut T\| -> String</code> | `x.to_string()`        | converts the [custom type] into a [string]                 |
| `to_debug`  | <code>\|x: &mut T\| -> String</code> | `format!("{:?}", x)`   | converts the [custom type] into a [string] in debug format |


The following functions are implemented using `to_string()` or `to_debug()` by default, but can be
overloaded with custom versions:

| Function      | Signature                                      | Default     | Usage                                                                  |
| ------------- | ---------------------------------------------- | ----------- | ---------------------------------------------------------------------- |
| `print`       | <code>\|x: &mut T\| -> String</code>           | `to_string` | converts the [custom type] into a [string] for the [`print`] statement |
| `debug`       | <code>\|x: &mut T\| -> String</code>           | `to_debug`  | converts the [custom type] into a [string] for the [`debug`] statement |
| `+` operator  | <code>\|s: &str, x: T\| -> String</code>       | `to_string` | concatenates the [custom type] with another [string]                   |
| `+` operator  | <code>\|x: &mut T, s: &str\| -> String</code>  | `to_string` | concatenates another [string] with the [custom type]                   |
| `+=` operator | <code>\|s: &mut ImmutableString, x: T\|</code> | `to_string` | appends the [custom type] to an existing [string]                      |
