Maximum Call Stack Depth
=======================

{{#include ../links.md}}

In Rhai, it is trivial for a function call to perform _infinite recursion_ such that all stack space
is exhausted.

```rust
// This is a function that, when called, recurses forever.
fn recurse_forever() {
    recurse_forever();
}
```

Rhai, by default, limits function calls to a maximum depth of 64 levels (8 levels in debug build).

This limit may be changed via the [`Engine::set_max_call_levels`][options] method.

A script exceeding the maximum call stack depth will terminate with an error result.

This check can be disabled via the [`unchecked`] feature for higher performance (but higher risks as well).

```rust
let mut engine = Engine::new();

engine.set_max_call_levels(10);     // allow only up to 10 levels of function calls

engine.set_max_call_levels(0);      // allow no function calls at all (max depth = zero)
```


```admonish info "Additional considerations"

When setting this limit, care must be also be taken to the evaluation depth of each _statement_
within a function.

It is entirely possible for a malicious script to embed a recursive call deep inside a nested
expression or statement block (see [maximum statement depth]).

~~~rust
fn bad_function(n) {
    // Bail out long before reaching the limit
    if n > 10 {
        return;
    }

    // Nest many, many levels deep...
    if check_1() {
        if check_2() {
            if check_3() {
                if check_4() {
                        :
                    if check_n() {
                        bad_function(n+1);  // <- recursive call!
                    }
                        :
                }
            }
        }
    }
}

// The function call below may still overflow the stack!
bad_function(0);
~~~
```
