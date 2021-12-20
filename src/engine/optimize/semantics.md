Subtle Semantic Changes After Optimization
=========================================

{{#include ../../links.md}}


Disappearing Runtime Errors
--------------------------

Some optimizations can alter subtle semantics of the script.

For example:

```rust no_run
if true {           // condition always true
    123.456;        // eliminated
    hello;          // eliminated, EVEN THOUGH the variable doesn't exist!
    foo(42)         // promoted up-level
}

foo(42)             // <- the above optimizes to this
```

If the original script were evaluated instead, it would have been an error &ndash;
the variable `hello` does not exist, so the script would have been terminated at that point
with a runtime error.

In fact, any errors inside a statement that has been eliminated will silently _disappear_.

```rust no_run
print("start!");
if my_decision { /* do nothing... */ }  // eliminated due to no effect
print("end!");

// The above optimizes to:

print("start!");
print("end!");
```

In the script above, if `my_decision` holds anything other than a boolean value,
the script should have been terminated due to a type error.

However, after optimization, the entire [`if`] statement is removed (because an access to
`my_decision` produces no side-effects), thus the script silently runs to completion without errors.


Eliminated Useless Work
-----------------------

Another example is more subtle &ndash; that of an empty loop body.

```rust no_run
// ... say, the 'Engine' is limited to no more than 10,000 operations...

// The following should fail because it exceeds the operations limit:
for n in 0..42000 {
    // empty loop
}

// The above is optimized away because the loop body is empty
// and the iterations simply do nothing.
()
```

Normally, and empty loop body inside a [`for`] statement with a pure iterator does nothing and can
be safely eliminated.

Thus the script now runs silently to completion without errors.

Without optimization, the script may fail by exceeding the [maximum number of operations] allowed.


Do Not Depend on Runtime Errors
------------------------------

Needless to say, it is usually a _Very Bad Idea™_ to depend on a script failing with a runtime error
or such kind of subtleties.

If it turns out to be necessary (why? I would never guess), turn script optimization off by setting
the optimization level to [`OptimizationLevel::None`].
