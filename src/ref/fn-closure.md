Closures
========

Many functions in the standard API expect [function pointer](fn-ptr.md) as parameters.

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

Sometimes it gets tedious to define separate [functions](functions.md) only to dispatch them via
single [function pointers](fn-ptr.md) &ndash; essentially, those [functions](functions.md) are only
ever called in one place.

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

Closures have a syntax similar to Rust's _closures_ (they are _not_ the same).

> `|`_param 1_`,` _param 2_`,` ... `,` _param n_`|` _statement_  
>
> `|`_param 1_`,` _param 2_`,` ... `,` _param n_`| {` _statements_... `}`  

No parameters:

> `||` _statement_  
>
> `|| {` _statements_... `}`


Rewrite Using Closures
----------------------

The above can be rewritten using closures.

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

Capture External Variables
--------------------------

~~~admonish tip.side "Tip: `is_shared`"

Use `is_shared` to check whether a particular dynamic value is shared.
~~~

Closures differ from standard functions because they can _captures_ [variables](variables.md) that
are not defined within the current scope, but are instead defined in an external scope &ndash; i.e.
where the it is created.

All [variables](variables.md) that are accessible during the time the closure is created are
automatically captured when they are used, as long as they are not shadowed by local
[variables](variables.md) defined within the function's.

The captured [variables](variables.md) are automatically converted into **reference-counted shared values**.

Therefore, similar to closures in many languages, these captured shared values persist through
reference counting, and may be read or modified even after the [variables](variables.md) that hold
them go out of scope and no longer exist.

```rust
let x = 1;                          // a normal variable

x.is_shared() == false;

let f = |y| x + y;                  // variable 'x' is auto-curried (captured) into 'f'

x.is_shared() == true;              // 'x' is now a shared value!

f.call(2) == 3;                     // 1 + 2 == 3

x = 40;                             // changing 'x'...

f.call(2) == 42;                    // the value of 'x' is 40 because 'x' is shared

// The above de-sugars into something like this:

fn anon_0001(x, y) { x + y }        // parameter 'x' is inserted

make_shared(x);                     // convert variable 'x' into a shared value

let f = anon_0001.curry(x);         // shared 'x' is curried
```


~~~admonish bug "Beware: Captured variables are truly shared"

The example below is a typical tutorial sample for many languages to illustrate the traps
that may accompany capturing external [variables](variables.md) in closures.

It prints `9`, `9`, `9`, ... `9`, `9`, not `0`, `1`, `2`, ... `8`, `9`, because there is ever only
_one_ captured [variable](variables.md), and all ten closures capture the _same_
[variable](variables.md).

```rust
let list = [];

for i in 0..10 {
    list.push(|| print(i));     // the for loop variable 'i' is captured
}

list.len() == 10;               // 10 closures stored in the array

list[0].type_of() == "Fn";      // make sure these are closures

for f in list {
    f.call();                   // all references to 'i' point to the same variable!
}
```
~~~

~~~admonish danger "Prevent data races"

Data races are possible in Rhai scripts.

Avoid performing a method call on a captured shared [variable](variables.md) (which essentially
takes a mutable reference to the shared object) while using that same [variable](variables.md) as a
parameter in the method call &ndash; this is a sure-fire way to generate a data race error.

If a shared value is used as the `this` pointer in a method call to a closure function,
then the same shared value _must not_ be captured inside that function, or a data race
will occur and the script will terminate with an error.

```rust
let x = 20;

x.is_shared() == false;         // 'x' not shared, so no data races

let f = |a| this += x + a;      // 'x' is captured in this closure

x.is_shared() == true;          // now 'x' is shared

x.call(f, 2);                   // <- error: data race detected on 'x'
```
~~~
