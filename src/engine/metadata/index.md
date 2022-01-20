Functions Metadata
==================

{{#include ../../links.md}}

The _metadata_ of a [function] means all relevant information related to a function's
definition including:

1. Its callable name

2. Its access mode (public or [private][`private`])

3. Its parameter names and types (if any)

4. Its return value and type (if any)

5. Its nature (i.e. native Rust or Rhai-scripted)

6. Its [namespace][function namespace] ([module] or global)

7. Its purpose, in the form of [doc-comments]

8. Usage notes, warnings, examples etc., in the form of [doc-comments]

A function's _signature_ encapsulates the first four pieces of information in a single concise line
of definition:

> `[private]` _name_ `(`_param 1_`:`_type 1_`,` _param 2_`:`_type 2_`,` ... `,` _param n_`:`_type n_`) ->` _return type_

Exporting metadata requires the [`metadata`] feature.
