Return Value
============

{{#include ../links.md}}

`return`
--------

The `return` statement is used to immediately stop evaluation and exist the current context
(typically a [function] call) yielding a _return value_.

```rust
return;             // equivalent to return ();

return 123 + 456;   // returns 579
```

A `return` statement at _global_ level stops the entire script evaluation,
the return value is taken as the result of the script evaluation.

A `return` statement inside a [function call][function] exits with a return value to the caller.


`exit`
------

Similar to the `return` statement, the `exit` _function_ is used to immediately stop evaluation,
but it does so regardless of where it is called from, even deep inside nested function calls.

```rust
fn foo() {
    exit(42);       // exit with result 42
}
fn bar() {
    foo();
}
fn baz() {
    bar();
}

let x = baz();      // exits with result 42

print(x);           // <- this is never run
```

The `exit` function is defined in the [`LanguageCorePackage`][built-in packages] but excluded when using a [raw `Engine`].

| Function | Parameter(s)              | Description                                                              |
| -------- | ------------------------- | ------------------------------------------------------------------------ |
| `exit`   | result value _(optional)_ | immediately terminate script evaluation (default result value is [`()`]) |
