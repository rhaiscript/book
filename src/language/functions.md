Functions
=========

{{#include ../links.md}}

Rhai supports defining functions in script (unless disabled with [`no_function`]):

```rust
fn add(x, y) {
    return x + y;
}

fn sub(x, y,) {     // trailing comma in parameters list is OK
    return x - y;
}

add(2, 3) == 5;

sub(2, 3,) == -1;   // trailing comma in arguments list is OK
```


Implicit Return
---------------

Just like in Rust, an implicit return can be used. In fact, the last statement of a block is _always_ the block's return value
regardless of whether it is terminated with a semicolon `';'`. This is different from Rust.

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
----------------------

Functions can only be defined at the global level, never inside a block or another function.

```rust
// Global level is OK
fn add(x, y) {
    x + y
}

// The following will not compile
fn do_addition(x) {
    fn add_y(n) {   // <- syntax error: functions cannot be defined inside another function
        n + y
    }

    add_y(x)
}
```


No Access to External Scope
--------------------------

Functions are not _closures_. They do not capture the calling environment
and can only access their own parameters.
They cannot access variables external to the function itself.

```rust
let x = 42;

fn foo() { x }          // <- syntax error: variable 'x' doesn't exist
```


But Can Call Other Functions
---------------------------

All functions in the same [`AST`] can call each other.

```rust
fn foo(x) { x + 1 }     // function defined in the global namespace

fn bar(x) { foo(x) }    // OK! function 'foo' can be called
```


Use Before Definition Allowed
----------------------------

Unlike C/C++, functions in Rhai can be defined _anywhere_ at global level.

A function does not need to be defined prior to being used in a script;
a statement in the script can freely call a function defined afterwards.

This is similar to Rust and many other modern languages, such as JavaScript's `function` keyword.


Arguments are Passed by Value
----------------------------

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


`this` &ndash; Simulating an Object Method
-----------------------------------------

Script-defined functions can also be called in method-call style.
When this happens, the keyword '`this`' binds to the object in the method call and can be changed.

```rust
fn change() {       // not that the object does not need a parameter
    this = 42;      // 'this' binds to the object in method-call
}

let x = 500;

x.change();         // call 'change' in method-call style, 'this' binds to 'x'

x == 42;            // 'x' is changed!

change();           // <- error: `this` is unbound
```


`is_def_fn`
-----------

Use `is_def_fn` to detect if a Rhai function is defined (and therefore callable),
based on its name and the number of parameters.

```rust
fn foo(x) { x + 1 }

is_def_fn("foo", 1) == true;

is_def_fn("foo", 0) == false;

is_def_fn("foo", 2) == false;

is_def_fn("bar", 1) == false;
```


Metadata
--------

The function `get_fn_metadata_list` is a _reflection_ API that returns an array of the metadata
of all script-defined functions in scope.

Functions from the following sources are returned, in order:

1) Encapsulated script environment (e.g. when loading a [module] from a script file),
2) Current script,
3) [Modules] imported via the [`import`] statement (latest imports first),
4) [Modules] added via [`Engine::register_static_module`]({{rootUrl}}/rust/modules/create.md) (latest registrations first)

The return value is an [array] of [object maps] (so `get_fn_metadata_list` is not available under
[`no_index`] or [`no_object`]), containing the following fields:

| Field          |         Type         | Optional? | Description                                                            |
| -------------- | :------------------: | :-------: | ---------------------------------------------------------------------- |
| `namespace`    |       [string]       |    yes    | the module _namespace_ if the function is defined within a module      |
| `access`       |       [string]       |    no     | `"public"` if the function is public,<br/>`"private"` if it is private |
| `name`         |       [string]       |    no     | function name                                                          |
| `params`       | [array] of [strings] |    no     | parameter names                                                        |
| `is_anonymous` |        `bool`        |    no     | is this function an anonymous function?                                |
