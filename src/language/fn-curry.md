Function Pointer Currying
=========================

{{#include ../links.md}}

```admonish info.side.wide "Automatic currying"

[Anonymous functions] defined via a closure syntax _capture_ external [variables]
that are not [shadowed][shadow] inside the [function's][function] scope.

This is accomplished via [automatic currying].
```

It is possible to _curry_ a [function pointer] by providing partial (or all) arguments.

Currying is done via the `curry` keyword and produces a new [function pointer] which carries
the curried arguments.

When the curried [function pointer] is called, the curried arguments are inserted starting from the _left_.

The actual call arguments should be reduced by the number of curried arguments.

```rust
fn mul(x, y) {                  // function with two parameters
    x * y
}

let func = mul;                 // <- de-sugars to 'Fn("mul")'

func.call(21, 2) == 42;         // two arguments are required for 'mul'

let curried = func.curry(21);   // currying produces a new function pointer which
                                // carries 21 as the first argument

let curried = curry(func, 21);  // function-call style also works

curried.call(2) == 42;          // <- de-sugars to 'func.call(21, 2)'
                                //    only one argument is now required
```
