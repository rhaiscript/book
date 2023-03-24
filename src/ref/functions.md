Functions
=========

Rhai supports defining functions in script via the `fn` keyword.

Valid function names are the same as valid [variable](variables.md) names.

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

~~~admonish tip.small "Tip: `is_def_fn`"

Use `is_def_fn` to detect if a Rhai function is defined (and therefore callable)
based on its name and the number of parameters (_arity_).

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

The last statement of a block is _always_ the block's return value regardless of whether it is
terminated with a semicolon `;`.

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

They cannot access [variables](variables.md) external to the function itself.

```rust
let x = 42;

fn foo() {
    x               // <- error: variable 'x' not found
}
```


But Can Call Other Functions and Access Modules
-----------------------------------------------

All functions can call each other.

```rust
fn foo(x) {         // function defined in the global namespace
    x + 1
}

fn bar(x) {
    foo(x)          // ok! function 'foo' can be called
}
```

In addition, [modules](modules/index.md) [imported](modules/import.md) at global level can be accessed.

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

When a [constant](constants.md) is declared at global scope, it is added to a special
[module](modules/index.md) called [`global`](global.md).

Functions can access those [constants](constants.md) via the special [`global`](global.md)
[module](modules/index.md).

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

Functions with the same name and same _number_ of parameters are equivalent.

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

```admonish warning.small "Rhai functions are pure"

The only possibility for a Rhai script-defined function to modify an external variable is
via the [`this`](fn-method.md) pointer.
```


Restrict the Type of `this`
---------------------------

```admonish tip.side "Tip: Automatically global"

Methods defined this way are automatically exposed to the global namespace.
```

In many cases where it is desired to implement methods for different custom types using
script-defined functions.

With a special syntax, it is possible to restrict calling a function only when the object pointed to
by [`this`](fn-method.md) is of a certain type:

> `fn`  _type name_ `.` _method_ `(` _parameters_ ... `)  {`  ...  `}`

or in quotes if the type name is not a valid identifier itself:

> `fn`  `"`_type name string_`"` `.` _method_ `(` _parameters_ ... `)  {`  ...  `}`

~~~admonish warning.small "Type name must be the same as `type_of`"

The _type name_ specified in front of the function name must match the output of [`type_of`](type-of.md)
for the required type.
~~~

~~~admonish tip.small "Tip: `int` and `float`"
`int` can be used in place of the system integer type (usually `i64` or `i32`).

`float` can be used in place of the system floating-point type (usually `f64` or `f32`).

Using these make scripts more portable.
~~~

### Examples

```js
/// 'my_method' can only be called on objects of type 'MyType' in method style
fn MyType.my_method(x, y) {
    this.update(x * y);
}

/// 'do_update' can only be called on objects of type 'Strange-Type#Name::with_!@#symbols'
/// in method style
fn "Strange-Type#Name::with_!@#symbols".do_update(x, y) {
    this.update(x * y);
}

/// Define a blanket version
fn do_update(x, y) {
    this += x * y;
}

/// 'mul_and_add' can only be called on integers in method style
fn int.mul_and_add(x, y) {
    this += x * y
}

let obj = create_my_type();

obj.type_of() == "MyType";

obj.my_method(42, 123);         // ok!

let x = 42;                     // x is an integer, not MyType

x.type_of() == "i64";

x.my_method(42, 123);           // <- error: 'my_method' not found

x.mul_and_add(42, 123);         // ok!

x.do_update(42, 123);           // <- this works because there is a blanket version
```
