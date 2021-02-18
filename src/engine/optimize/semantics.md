Subtle Semantic Changes After Optimization
=========================================

{{#include ../../links.md}}

Some optimizations can alter subtle semantics of the script.

For example:

```rust,no_run
if true {           // condition always true
    123.456;        // eliminated
    hello;          // eliminated, EVEN THOUGH the variable doesn't exist!
    foo(42)         // promoted up-level
}

foo(42)             // <- the above optimizes to this
```

If the original script were evaluated instead, it would have been an error &ndash; the variable `hello` does not exist,
so the script would have been terminated at that point with an error return.

In fact, any errors inside a statement that has been eliminated will silently _disappear_:

```rust,no_run
print("start!");
if my_decision { /* do nothing... */ }  // eliminated due to no effect
print("end!");

// The above optimizes to:

print("start!");
print("end!");
```

In the script above, if `my_decision` holds anything other than a boolean value,
the script should have been terminated due to a type error.

However, after optimization, the entire `if` statement is removed (because an access to `my_decision` produces
no side-effects), thus the script silently runs to completion without errors.

It is usually a _Very Bad Idea™_ to depend on a script failing or such kind of subtleties, but if it turns out to be necessary
(why? I would never guess), turn script optimization off by setting the optimization level to [`OptimizationLevel::None`].
