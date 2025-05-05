Printing for Custom Types
=========================

{{#include ../links.md}}


Provide These Functions
-----------------------

To use [custom types] for [`print`] and [`debug`], or convert a [custom type] into a [string],
it is necessary that the following functions, at minimum, be registered (assuming the [custom type]
is `T: Display + Debug`).

| Function    | Signature                 | Typical implementation | Usage                                                      |
| ----------- | ------------------------- | ---------------------- | ---------------------------------------------------------- |
| `to_string` | `\|x: &mut T\| -> String` | `x.to_string()`        | converts the [custom type] into a [string]                 |
| `to_debug`  | `\|x: &mut T\| -> String` | `format!("{x:?}")`     | converts the [custom type] into a [string] in debug format |

~~~admonish tip.small "Tip: `#[rhai_fn(global)]`"

If these functions are defined via a [plugin module], be sure to include the `#[rhai_fn(global)]` attribute
in order to make them available globally.

See [this section]({{rootUrl}}/plugins/module.md#use-rhai_fnglobal) for more details.
~~~


Also Consider These
-------------------

The following functions are implemented using `to_string` or `to_debug` by default, but can be
overloaded with custom versions.

| Function      | Signature                           | Default     | Usage                                                                  |
| ------------- | ----------------------------------- | ----------- | ---------------------------------------------------------------------- |
| `print`       | `\|x: &mut T\| -> String`           | `to_string` | converts the [custom type] into a [string] for the [`print`] statement |
| `debug`       | `\|x: &mut T\| -> String`           | `to_debug`  | converts the [custom type] into a [string] for the [`debug`] statement |
| `+` operator  | `\|s: &str, x: T\| -> String`       | `to_string` | concatenates the [custom type] with another [string]                   |
| `+` operator  | `\|x: &mut T, s: &str\| -> String`  | `to_string` | concatenates another [string] with the [custom type]                   |
| `+=` operator | `\|s: &mut ImmutableString, x: T\|` | `to_string` | appends the [custom type] to an existing [string]                      |
