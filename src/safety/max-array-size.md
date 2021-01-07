Maximum Size of Arrays
=====================

{{#include ../links.md}}

Limit How Large Arrays Can Grow
------------------------------

Rhai by default does not limit how large an [array] can be.

This can be changed via the `Engine::set_max_array_size` method, with zero being unlimited (the default).

A script attempting to create an array literal larger than the maximum will terminate with a parse error.

Any script operation that produces an array larger than the maximum also terminates the script with an error result.

This check can be disabled via the [`unchecked`] feature for higher performance (but higher risks as well).

```rust
let mut engine = Engine::new();

engine.set_max_array_size(500); // allow arrays only up to 500 items

engine.set_max_array_size(0);   // allow unlimited arrays
```


Setting Maximum Size
-------------------

Be conservative when setting a maximum limit and always consider the fact that a registered function may grow
an array's size without Rhai noticing until the very end.

For instance, the built-in '`+`' operator for arrays concatenates two arrays together to form one larger array;
if both arrays are _slightly_ below the maximum size limit, the resultant array may be almost _twice_ the maximum size.

As a malicious script may create a deeply-nested array which consumes huge amounts of memory while each individual
array still stays under the maximum size limit, Rhai also recursively adds up the sizes of all [strings], [arrays]
and [object maps] contained within each array to make sure that the _aggregate_ sizes of none of these data structures
exceed their respective maximum size limits (if any).
