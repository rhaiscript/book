Function Pointers
=================

It is possible to store a _function pointer_ in a variable just like a normal value.

A function pointer is created via the `Fn` function, which takes a [string](strings-chars.md) parameter.

Call a function pointer via the `call` method.


Short-Hand Notation
-------------------

```admonish warning.side "Not for native"

Native Rust functions cannot use this short-hand notation.
```

Having to write `Fn("foo")` in order to create a function pointer to the [function](functions.md)
`foo` is a chore, so there is a short-hand available.

A function pointer to any _script-defined_ [function](functions.md) _within the same script_ can be
obtained simply by referring to the [function's](functions.md) name.

```rust
fn foo() { ... }        // function definition

let f = foo;            // function pointer to 'foo'

let f = Fn("foo");      // <- the above is equivalent to this

let g = bar;            // error: variable 'bar' not found
```


Built-in Functions
------------------

The following standard methods operate on function pointers.

| Function                           | Parameter(s) | Description                                                                                  |
| ---------------------------------- | ------------ | -------------------------------------------------------------------------------------------- |
| `name` method and property         | _none_       | returns the name of the [function](functions.md) encapsulated by the function pointer        |
| `is_anonymous` method and property | _none_       | does the function pointer refer to an [anonymous function](fn-anon.md)?                      |
| `call`                             | _arguments_  | calls the [function](functions.md) matching the function pointer's name with the _arguments_ |


Examples
--------

```rust
fn foo(x) { 41 + x }

let func = Fn("foo");       // use the 'Fn' function to create a function pointer

let func = foo;             // <- short-hand: equivalent to 'Fn("foo")'

print(func);                // prints 'Fn(foo)'

let func = fn_name.Fn();    // <- error: 'Fn' cannot be called in method-call style

func.type_of() == "Fn";     // type_of() as function pointer is 'Fn'

func.name == "foo";

func.call(1) == 42;         // call a function pointer with the 'call' method

foo(1) == 42;               // <- the above de-sugars to this

call(func, 1);              // normal function call style also works for 'call'

let len = Fn("len");        // 'Fn' also works with registered native Rust functions

len.call("hello") == 5;

let fn_name = "hello";      // the function name does not have to exist yet

let hello = Fn(fn_name + "_world");

hello.call(0);              // error: function not found - 'hello_world (i64)'
```


Warning &ndash; Global Namespace Only
-------------------------------------

Because of their dynamic nature, function pointers cannot refer to functions in
[`import`](modules/import.md)-ed [modules](modules/index.md).

They can only refer to [functions](functions.md) defined globally within the script
or a built-in function.

```js
import "foo" as f;          // assume there is 'f::do_work()'

f::do_work();               // works!

let p = Fn("f::do_work");   // error: invalid function name

fn do_work_now() {          // call it from a local function
    f::do_work();
}

let p = Fn("do_work_now");

p.call();                   // works!
```


Dynamic Dispatch
----------------

The purpose of function pointers is to enable rudimentary _dynamic dispatch_, meaning to determine,
at runtime, which function to call among a group.

Although it is possible to simulate dynamic dispatch via a number and a large
[`if-then-else-if`](if.md) statement, using function pointers significantly simplifies the code.

```rust
let x = some_calculation();

// These are the functions to call depending on the value of 'x'
fn method1(x) { ... }
fn method2(x) { ... }
fn method3(x) { ... }

// Traditional - using decision variable
let func = sign(x);

// Dispatch with if-statement
if func == -1 {
    method1(42);
} else if func == 0 {
    method2(42);
} else if func == 1 {
    method3(42);
}

// Using pure function pointer
let func = if x < 0 {
    method1
} else if x == 0 {
    method2
} else if x > 0 {
    method3
};

// Dynamic dispatch
func.call(42);

// Using functions map
let map = [ method1, method2, method3 ];

let func = sign(x) + 1;

// Dynamic dispatch
map[func].call(42);
```


Bind the `this` Pointer
-----------------------

When `call` is called as a _method_ but not on a function pointer, it is possible to dynamically dispatch
to a function call while binding the object in the method call to the `this` pointer of the function.

To achieve this, pass the function pointer as the _first_ argument to `call`:

```rust
fn add(x) {                 // define function which uses 'this'
    this += x;
}

let func = add;             // function pointer to 'add'

func.call(1);               // error: 'this' pointer is not bound

let x = 41;

func.call(x, 1);            // error: function 'add (i64, i64)' not found

call(func, x, 1);           // error: function 'add (i64, i64)' not found

x.call(func, 1);            // 'this' is bound to 'x', dispatched to 'func'

x == 42;
```

Beware that this only works for [_method-call_](fn-method.md) style.
Normal function-call style cannot bind the `this` pointer (for syntactic reasons).
