Functions Metadata
==================

{{#include ../../links.md}}

The _metadata_ of a [function] means all relevant information related to a function's
definition including:

1. Its callable name

2. Its access mode (public or [private][`private`])

3. Its parameters and types (if any)

4. Its return value and type (if any)

5. Its nature (i.e. native Rust-based or Rhai script-based)

6. Its [namespace][function namespace] (module or global)

7. Its purpose, in the form of [doc-comments]

8. Usage notes, warnings, etc., in the form of [doc-comments]

A function's _signature_ encapsulates the first four pieces of information in a single
concise line of definition:

> `[private] fn_name ( param_1: type_1, param_2: type_2, ... , param_n : type_n ) -> return_type`
