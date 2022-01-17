Disable Custom Types
====================

{{#include ../links.md}}

The [`no_object`] feature disables support for [custom types] including:

* [_method-style_]({{rootUrl}}/rust/methods.md}}) function calls (e.g. `obj.method()`),

* [object maps] and the [`Map`] type,

* the `register_get`, `register_get_result`, `register_set`, `register_set_result` and `register_get_set` API's for [`Engine`]
