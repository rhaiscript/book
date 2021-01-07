Checked Arithmetic
=================

{{#include ../links.md}}

By default, all arithmetic calculations in Rhai are _checked_, meaning that the script terminates
with an error whenever it detects a numeric over-flow/under-flow condition or an invalid
floating-point operation, instead of crashing the entire system.

This checking can be turned off via the [`unchecked`] feature for higher performance
(but higher risks as well).
