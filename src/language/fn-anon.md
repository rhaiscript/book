Anonymous Functions
===================

{{#include ../links.md}}

Sometimes it gets tedious to define separate functions only to dispatch them via single [function pointers].
This scenario is especially common when simulating object-oriented programming ([OOP]).

```rust no_run
// Define object
let obj = #{
    data: 42,
    increment: Fn("inc_obj"),           // use function pointers to
    decrement: Fn("dec_obj"),           // refer to method functions
    print: Fn("print_obj")
};

// Define method functions one-by-one
fn inc_obj(x) { this.data += x; }
fn dec_obj(x) { this.data -= x; }
fn print_obj() { print(this.data); }
```

The above can be replaced by using _anonymous functions_ which have the same syntax as Rust's closures
(but they are **NOT** real closures, merely syntactic sugar).

```rust no_run
let obj = #{
    data: 42,
    increment: |x| this.data += x,      // one-liner
    decrement: |x| this.data -= x,
    print_obj: || {
        print(this.data);               // full function body
    }
};

// The above is equivalent to...

let obj = #{
    data: 42,
    increment: Fn("anon_fn_0001"),
    decrement: Fn("anon_fn_0002"),
    print: Fn("anon_fn_0003")
};

fn anon_fn_0001(x) {
    this.data += x;                     // when called, 'this' maps to the object map
}
fn anon_fn_0002(x) {
    this.data -= x;                     // when called, 'this' maps to the object map
}
fn anon_fn_0003() {
    print(this.data);                   // when called, 'this' maps to the object map
}
```

Anonymous functions can be disabled via [`Engine::set_allow_anonymous_function`][options].


WARNING &ndash; NOT Real Closures
--------------------------------

Remember: anonymous functions, though having the same syntax as Rust _closures_, are themselves
**NOT** real closures.

In particular, they capture their execution environment via [automatic currying]
(disabled via [`no_closure`]).
