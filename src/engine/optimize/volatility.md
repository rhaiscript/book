Volatility Considerations for Full Optimization Level
=====================================================

{{#include ../../links.md}}

Even if a custom function does not mutate state nor cause side-effects, it may still be _volatile_,
i.e. it _depends_ on external environment and does not guarantee the same result for the same inputs.

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

```admonish danger.small "Warning"

**Avoid using [`OptimizationLevel::Full`]** if volatile custom functions are involved.
```

The optimizer, when using [`OptimizationLevel::Full`], _merrily assumes_ that all functions are
_non-volatile_, so when it finds [constant] arguments (or none) it eagerly executes the function
call and replaces it with the result.

This causes the script to behave differently from the intended semantics.

~~~admonish tip "Tip: Mark a function as volatile"

All native functions are assumed to be **non-volatile**, meaning that they are eagerly called under
[`OptimizationLevel::Full`] when all arguments are [constant] (or none).

It is possible to [mark a function defined within a plugin module as volatile]({{rootUrl}}/plugins/module.md#volatile-functions)
to prevent this behavior.

```rust
#[export_module]
mod my_module {
    // This function is marked 'volatile' and will not be
    // eagerly executed even under OptimizationLevel::Full.
    #[rhai_fn(volatile)]
    pub get_current_time(am_pm: bool) -> String {
        // ...
    }
}
```
~~~
