Return Value
============

`return`
--------

The `return` statement is used to immediately stop evaluation and exist the current context
(typically a [function](functions.md) call) yielding a _return value_.

```rust
return;             // equivalent to return ();

return 123 + 456;   // returns 579
```

A `return` statement at _global_ level exits the script with the return value as the result.

A `return` statement inside a [function call](functions.md) exits with a return value to the caller.


`exit`
------

Similar to the `return` statement, the `exit` _function_ is used to immediately stop evaluation,
but it does so regardless of where it is called from, even deep inside nested function calls.

The result value of `exit`, when omitted, defaults to `()`.

```rust
fn foo() {
    exit(42);       // exit with result value 42
}
fn bar() {
    foo();
}
fn baz() {
    bar();
}

let x = baz();      // exits with result value 42

print(x);           // <- this is never run
```
