Capture The Calling Scope for Function Call
==========================================

{{#include ../links.md}}


Peeking Out of The Pure Box
---------------------------

Rhai functions are _pure_, meaning that they depend on on their arguments and have no
access to the calling environment.

When a function accesses a variable that is not defined within that function's scope,
it raises an evaluation error.

It is possible, through a special syntax, to capture the calling scope &ndash; i.e. the scope
that makes the function call &ndash; and access variables defined there.

```rust no_run
fn foo(y) {             // function accesses 'x' and 'y', but 'x' is not defined
    x += y;             // 'x' is modified in this function
    x
}

let x = 1;

foo(41);                // error: variable 'x' not found

// Calling a function with a '!' causes it to capture the calling scope

foo!(41) == 42;         // the function can access the value of 'x', but cannot change it

x == 1;                 // 'x' is still the original value

x.method!();            // <- syntax error: capturing is not allowed in method-call style

// Capturing also works for function pointers

let f = Fn("foo");

call!(f, 41) == 42;     // must use function-call style

f.call!(41);            // <- syntax error: capturing is not allowed in method-call style

// Capturing is not available for module functions

import "hello" as h;

h::greet!();            // <- syntax error: capturing is not allowed in namespace-qualified calls
```


No Mutations
------------

Variables in the calling scope are captured as cloned copies.
Changes to them do **not** reflect back to the calling scope.

Rhai functions remain _pure_ in the sense that they can never mutate their environment.


Caveat Emptor
-------------

Functions relying on the calling scope is often a _Very Bad Idea™_ because it makes code
almost impossible to reason and maintain, as their behaviors are volatile and unpredictable.

They behave more like macros that are expanded inline than actual function calls, thus the
syntax is also similar to Rust's macro invocations.

This usage should be at the last resort. YOU HAVE BEEN WARNED.
