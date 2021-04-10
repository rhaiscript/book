Maximum Length of Strings
========================

{{#include ../links.md}}

Limit How Long Strings Can Grow
------------------------------

Rhai by default does not limit how long a [string] can be.

This can be changed via the `Engine::set_max_string_size` method, with zero being unlimited (the default).

A script attempting to create a string literal longer than the maximum length will terminate with a parse error.

Any script operation that produces a string longer than the maximum also terminates the script with an error result.

This check can be disabled via the [`unchecked`] feature for higher performance (but higher risks as well).

```rust , no_run
let mut engine = Engine::new();

engine.set_max_string_size(500);    // allow strings only up to 500 bytes long (in UTF-8 format)

engine.set_max_string_size(0);      // allow unlimited string length
```


Setting Maximum Length
---------------------

Be conservative when setting a maximum limit and always consider the fact that a registered function may grow
a string's length without Rhai noticing until the very end.

For instance, the built-in `+` operator for strings concatenates two strings together to form one longer string;
if both strings are _slightly_ below the maximum length limit, the resultant string may be almost _twice_ the maximum length.

