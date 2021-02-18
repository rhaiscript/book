Function Pointer Currying
========================

{{#include ../links.md}}

It is possible to _curry_ a [function pointer] by providing partial (or all) arguments.

Currying is done via the `curry` keyword and produces a new [function pointer] which carries
the curried arguments.

When the curried [function pointer] is called, the curried arguments are inserted starting from the left.
The actual call arguments should be reduced by the number of curried arguments.

```rust,no_run
fn mul(x, y) {                  // function with two parameters
    x * y
}

let func = Fn("mul");

func.call(21, 2) == 42;         // two arguments are required for 'mul'

let curried = func.curry(21);   // currying produces a new function pointer which
                                // carries 21 as the first argument

let curried = curry(func, 21);  // function-call style also works

curried.call(2) == 42;          // <- de-sugars to 'func.call(21, 2)'
                                //    only one argument is now required
```


Automatic Currying
------------------

[Anonymous functions] defined via a closure syntax _capture_ external variables
that are not shadowed inside the function's scope.

This is accomplished via [automatic currying].
