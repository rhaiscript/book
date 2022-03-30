Volatility Considerations for Full Optimization Level
===================================================

{{#include ../../links.md}}

Even if a custom function does not mutate state nor cause side-effects, it may still be _volatile_,
i.e. it _depends_ on the external environment and is not _pure_.

A perfect example is a function that gets the current time &ndash; obviously each run will return a
different value!

```rust
print(get_current_time(true));      // prints the current time
                                    // notice the call to 'get_current_time'
                                    // has constant arguments

// The above, under full optimization level, is rewritten to:

print("10:25AM");                   // the function call is replaced by
                                    // its result at the time of optimization!
```

The optimizer, when using [`OptimizationLevel::Full`], _merrily assumes_ that all functions are
_non-volatile_, so when it finds [constant] arguments (or none) it eagerly executes the function
call and replaces it with the result.

This causes the script to behave differently from the intended semantics.

```admonish danger.small "Warning"

**Avoid using [`OptimizationLevel::Full`]** if volatile custom functions are involved.
```
