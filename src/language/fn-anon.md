Anonymous Functions
===================

{{#include ../links.md}}

Many functions in the standard API expect [function pointer] as parameters.

For example:

```rust
// Function 'double' defined here - used only once
fn double(x) { 2 * x }

// Function 'square' defined here - again used only once
fn square(x) { x * x }

let x = [1, 2, 3, 4, 5];

// Pass a function pointer to 'double'
let y = x.map(double);

// Pass a function pointer to 'square' using Fn(...) notation
let z = y.map(Fn("square"));
```

Sometimes it gets tedious to define separate [functions] only to dispatch them via single [function pointers] &ndash;
essentially, those [functions] are only ever called in one place.

This scenario is especially common when simulating object-oriented programming ([OOP]).

```rust
// Define functions one-by-one
fn obj_inc(x, y) { this.data += x * y; }
fn obj_dec(x) { this.data -= x; }
fn obj_print() { print(this.data); }

// Define object
let obj = #{
    data: 42,
    increment: obj_inc,     // use function pointers to
    decrement: obj_dec,     // refer to method functions
    print: obj_print
};
```


Syntax
------

Anonymous [functions] have a syntax similar to Rust's _closures_ (they are _not_ the same).

> `|`_param 1_`,` _param 2_`,` ... `,` _param n_`|` _statement_  
>
> `|`_param 1_`,` _param 2_`,` ... `,` _param n_`| {` _statements_... `}`  

No parameters:

> `||` _statement_  
>
> `|| {` _statements_... `}`

Anonymous functions can be disabled via [`Engine::set_allow_anonymous_function`][options].


Rewrite Using Anonymous Functions
---------------------------------

The above can be rewritten using _anonymous [functions]_.

```rust
let x = [1, 2, 3, 4, 5];

let y = x.map(|x| 2 * x);

let z = y.map(|x| x * x);

let obj = #{
    data: 42,
    increment: |x, y| this.data += x * y,   // one statement
    decrement: |x| this.data -= x,          // one statement
    print_obj: || {
        print(this.data);                   // full function body
    }
};
```

This de-sugars to:

```rust
// Automatically generated...
fn anon_fn_0001(x) { 2 * x }
fn anon_fn_0002(x) { x * x }
fn anon_fn_0003(x, y) { this.data += x * y; }
fn anon_fn_0004(x) { this.data -= x; }
fn anon_fn_0005() { print(this.data); }

let x = [1, 2, 3, 4, 5];

let y = x.map(anon_fn_0001);

let z = y.map(anon_fn_0002);

let obj = #{
    data: 42,
    increment: anon_fn_0003,
    decrement: anon_fn_0004,
    print: anon_fn_0005
};
```

```admonish danger.small "NOT real closures"

Remember: though having the same syntax as Rust _closures_, anonymous functions are themselves
**NOT** real closures.

In particular, they capture their execution environment via [automatic currying]
(disabled via [`no_closure`]).
```
