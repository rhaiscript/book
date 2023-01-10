Functions
=========

{{#include ../links.md}}

Rhai supports defining functions in script, with a syntax that is very similar to Rust without types.

Valid function names are the same as valid [variable] names.

```rust
fn add(x, y) {
    x + y
}

fn sub(x, y,) {     // trailing comma in parameters list is OK
    x - y
}

add(2, 3) == 5;

sub(2, 3,) == -1;   // trailing comma in arguments list is OK
```

```admonish tip.small "Tip: Disable functions"

Defining functions can be disabled via the [`no_function`] feature.
```

~~~admonish tip.small "Tip: `is_def_fn`"

Use `is_def_fn` (not available under [`no_function`]) to detect if a Rhai function is defined
(and therefore callable) based on its name and the number of parameters (_arity_).

```rust
fn foo(x) { x + 1 }

is_def_fn("foo", 1) == true;

is_def_fn("foo", 0) == false;

is_def_fn("foo", 2) == false;

is_def_fn("bar", 1) == false;
```
~~~


Implicit Return
---------------

Just like in Rust, an implicit return can be used. In fact, the last statement of a block is
_always_ the block's return value regardless of whether it is terminated with a semicolon `;`.
This is different from Rust.

```rust
fn add(x, y) {      // implicit return:
    x + y;          // value of the last statement (no need for ending semicolon)
                    // is used as the return value
}

fn add2(x) {
    return x + 2;   // explicit return
}

add(2, 3) == 5;

add2(42) == 44;
```


Global Definitions Only
-----------------------

Functions can only be defined at the global level, never inside a block or another function.

Again, this is different from Rust.

```rust
// Global level is OK
fn add(x, y) {
    x + y
}

// The following will not compile
fn do_addition(x) {
    fn add_y(n) {   // <- syntax error:  cannot define inside another function
        n + y
    }

    add_y(x)
}
```


No Access to External Scope
---------------------------

Functions are not _closures_. They do not capture the calling environment and can only access their
own parameters.

They cannot access [variables] external to the function itself.

```rust
let x = 42;

fn foo() {
    x               // <- error: variable 'x' not found
}
```


But Can Call Other Functions and Access Modules
-----------------------------------------------

All functions in the same [`AST`] can call each other.

```rust
fn foo(x) {         // function defined in the global namespace
    x + 1
}

fn bar(x) {
    foo(x)          // ok! function 'foo' can be called
}
```

In addition, [modules] [imported][`import`] at global level can be accessed.

```js
import "hello" as hey;
import "world" as woo;

{
    import "x" as xyz;  // <- this module is not at global level
}                       // <- it goes away here

fn foo(x) {
    hey::process(x);    // ok! imported module 'hey' can be accessed

    print(woo::value);  // ok! imported module 'woo' can be accessed

    xyz::do_work();     // <- error: module 'xyz' not found
}
```


Automatic Global Module
-----------------------

When a [constant] is declared at global scope, it is added to a special [module] called [`global`].

Functions can access those [constants] via the special [`global`] [module].

Naturally, the automatic [`global`] [module] is not available under [`no_function`] nor [`no_module`].

```rust
const CONSTANT = 42;        // this constant is automatically added to 'global'

let hello = 1;              // variables are not added to 'global'

{
    const INNER = 0;        // this constant is not at global level
}                           // <- it goes away here

fn foo(x) {
    x * global::hello       // <- error: variable 'hello' not found in 'global'

    x * global::CONSTANT    // ok! 'CONSTANT' exists in 'global'

    x * global::INNER       // <- error: constant 'INNER' not found in 'global'
}
```


Use Before Definition Allowed
-----------------------------

Unlike C/C++, functions in Rhai can be defined _anywhere_ at global level.

A function does not need to be defined prior to being used in a script; a statement in the script
can freely call a function defined afterwards.

This is similar to Rust and many other modern languages, such as JavaScript's `function` keyword.

```rust
let x = foo(41);    // <- I can do this!

fn foo(x) {         // <- define 'foo' after use
    x + 1
}
```


Arguments are Passed by Value
-----------------------------

Functions defined in script always take [`Dynamic`] parameters (i.e. they can be of any types).
Therefore, functions with the same name and same _number_ of parameters are equivalent.

All arguments are passed by _value_, so all Rhai script-defined functions are _pure_
(i.e. they never modify their arguments).

Any update to an argument will **not** be reflected back to the caller.

```rust
fn change(s) {      // 's' is passed by value
    s = 42;         // only a COPY of 's' is changed
}

let x = 500;

change(x);

x == 500;           // 'x' is NOT changed!
```
