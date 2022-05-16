Return Value
============

The `return` statement is used to immediately stop evaluation and exist the current context
(typically a [function](functions.md) call) yielding a _return value_.

```rust
return;             // equivalent to return ();

return 123 + 456;   // returns 579
```

A `return` statement at _global_ level exits the script with the return value as the result.

A `return` statement inside a [function call](functions.md) exits with a return value to the caller.
