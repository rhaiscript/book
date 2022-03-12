Maximum Number of Operations
===========================

{{#include ../links.md}}


In Rhai, it is trivial to construct _infinite loops_, or scripts that run for a very long time.

```rust,no_run
loop { ... }                        // infinite loop

while 1 < 2 { ... }                 // loop with always-true condition
```

Rhai by default does not limit how much time or CPU a script consumes.

This can be changed via the [`Engine::set_max_operations`][options] method, with zero being unlimited (the default).

The _operations count_ is intended to be a very course-grained measurement of the amount of CPU that a script
has consumed, allowing the system to impose a hard upper limit on computing resources.

A script exceeding the maximum operations count terminates with an error result.
This can be disabled via the [`unchecked`] feature for higher performance (but higher risks as well).

```rust,no_run
let mut engine = Engine::new();

engine.set_max_operations(500);     // allow only up to 500 operations for this script

engine.set_max_operations(0);       // allow unlimited operations
```


```admonish question.small "What does one _operation_ mean?"

The concept of one single _operation_ in Rhai is volatile &ndash; it roughly equals one expression
node, loading one [variable]/[constant], one [operator] call, one iteration of a loop, or one
[function] call etc. with sub-expressions, statements and [function] calls executed inside these
contexts accumulated on top.

A good rule-of-thumb is that one simple non-trivial expression consumes on average 5-10 operations.

One _operation_ can take an unspecified amount of time and real CPU cycles, depending on the particulars.
For example, loading a [constant] consumes very few CPU cycles, while calling an external Rust function,
though also counted as only one operation, may consume much more computing resources.

To help visualize, think of an _operation_ as roughly equals to one _instruction_ of a hypothetical CPU
which includes _specialized_ instructions, such as _function call_, _load module_ etc., each taking up
one CPU cycle to execute.
```
