Statement Expression
====================

{{#include ../links.md}}

```admonish warning.side "Differs from Rust"

This is different from Rust where, if the last statement is terminated by a semicolon, the block's
return value defaults to `()`.
```

Like Rust, a statement can be used anywhere where an _expression_ is expected.

These are called, for lack of a more creative name, "statement expressions."

The _last_ statement of a statement block is _always_ the block's return value when used as a statement,
_regardless_ of whether it is terminated by a semicolon or not.

If the last statement has no return value (e.g. variable definitions, assignments) then it is
assumed to be [`()`].

```rust
let x = {
    let foo = calc_something();
    let bar = foo + baz;
    bar.further_processing();       // <- this is the return value
};                                  // <- semicolon is needed here...

// The above is equivalent to:
let result;
{
    let foo = calc_something();
    let bar = foo + baz;
    result = bar.further_processing();
}
let x = result;

// Statement expressions can be inserted inside normal expressions
// to avoid duplicated calculations
let x = foo(bar) + { let v = calc(); process(v, v.len, v.abs) } + baz;

// The above is equivalent to:
let foo_result = foo(bar);
let calc_result;
{
    let v = calc();
    result = process(v, v.len, v.abs);  // <- avoid calculating 'v'
}
let x = foo_result + calc_result + baz;

// Statement expressions are also useful as function call arguments
// when side effects are desired
do_work(x, y, { let z = foo(x, y); print(z); z });
           // ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
           //       statement expression
```

Statement expressions can be disabled via [`Engine::set_allow_statement_expression`][options].
