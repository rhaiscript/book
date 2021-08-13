Return Values
=============

{{#include ../links.md}}

The `return` statement is used to immediately stop evaluation and exist the current context
(typically a function call) yielding a _return value_.

```rust no_run
return;             // equivalent to return ();

return 123 + 456;   // returns 579
```

A `return` statement at _global_ level stop the entire script evaluation,
the return value is taken as the result of the script evaluation.
