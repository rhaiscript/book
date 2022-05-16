Function Pointer Currying
=========================

```admonish info.side.wide "Automatic currying"

[Anonymous functions](fn-anon.md) defined via a closure syntax _capture_ external [variables](variables.md)
that are not [shadowed](shadow.md) inside the [function's](functions.md) scope.

This is accomplished via [automatic currying](fn-closure.md).
```

It is possible to _curry_ a [function pointer](fn-ptr.md) by providing partial (or all) arguments.

Currying is done via the `curry` keyword and produces a new [function pointer](fn-ptr.md) which
carries the curried arguments.

When the curried [function pointer](fn-ptr.md) is called, the curried arguments are inserted
starting from the _left_.

The actual call arguments should be reduced by the number of curried arguments.

```rust
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
