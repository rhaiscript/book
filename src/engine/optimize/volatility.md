Volatility Considerations for Full Optimization Level
===================================================

{{#include ../../links.md}}

Even if a custom function does not mutate state nor cause side-effects, it may still be _volatile_,
i.e. it _depends_ on the external environment and is not _pure_.

A perfect example is a function that gets the current time &ndash; obviously each run will return a different value!

The optimizer, when using [`OptimizationLevel::Full`], _merrily assumes_ that all functions are _non-volatile_,
so when it finds [constant] arguments (or none) it eagerly executes the function call and replaces it with the result.

This causes the script to behave differently from the intended semantics.

Therefore, **avoid using [`OptimizationLevel::Full`]** if volatile custom functions are involved.
