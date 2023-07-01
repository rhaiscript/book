Maximum Call Stack Depth
========================

{{#include ../links.md}}

In Rhai, it is trivial for a function call to perform _infinite recursion_ (or a very deeply-nested
recursion) such that all stack space is exhausted.

```rust
// This is a function that, when called, recurses forever.
fn recurse_forever() {
    recurse_forever();
}
```

```admonish info.side "Main stack size"

The main stack-size of a program is _not_ determined by Rust but is platform-dependent.

See [this on-line Rust docs](https://doc.rust-lang.org/std/thread/#stack-size) for more details.
```

Because of its intended embedded usage, Rhai, by default, limits function calls to a maximum depth
of 64 levels (8 levels in debug build) in order to fit into most platforms' default stack sizes.

This limit may be changed via the [`Engine::set_max_call_levels`][options] method.

A script exceeding the maximum call stack depth will terminate with an error result.

This check can be disabled via the [`unchecked`] feature for higher performance (but higher risks as well).

```rust
let mut engine = Engine::new();

engine.set_max_call_levels(10);     // allow only up to 10 levels of function calls

engine.set_max_call_levels(0);      // allow no function calls at all (max depth = zero)
```


```admonish info.small "Additional considerations"

When setting this limit, care must be also be taken to the evaluation depth of each _statement_
within a function.

It is entirely possible for a malicious script to embed a recursive call deep inside a nested
expression or statements block (see [maximum statement depth]).

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

```admonish tip.small "Tip: Getting around the stack size limit"

While the stack size of a program's _main_ thread is platform-specific, Rust defaults to a stack
size of 2MB for spawned threads.

This default can further be changed such that a spawned thread has as large a stack as needed.

See [the on-line Rust docs](https://doc.rust-lang.org/std/thread/#stack-size) for more details.

Therefore, in order to relax the stack size limit for scripts, run the [`Engine`] in a separate
spawned thread with a larger stack.
```
