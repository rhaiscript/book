Function Pointers
=================

{{#include ../links.md}}

```admonish question.side "Trivia"

A function pointer simply stores the _name_ of the [function] as a [string].
```

It is possible to store a _function pointer_ in a variable just like a normal value.

A function pointer is created via the `Fn` [function], which takes a [string] parameter.

Call a function pointer via the `call` method.


Short-Hand Notation
-------------------

```admonish warning.side "Not for native"

Native Rust functions cannot use this short-hand notation.
```

Having to write `Fn("foo")` in order to create a function pointer to the [function] `foo` is a chore,
so there is a short-hand available.

A function pointer to any _script-defined_ [function] _within the same script_ can be obtained simply
by referring to the [function's][function] name.

```rust
fn foo() { ... }        // function definition

let f = foo;            // function pointer to 'foo'

let f = Fn("foo");      // <- the above is equivalent to this

let g = bar;            // error: variable 'bar' not found
```

The short-hand notation is particularly useful when passing [functions] as [closure] arguments.

```rust
fn is_even(n) { n % 2 == 0 }

let array = [1, 2, 3, 4, 5];

array.filter(is_even);

array.filter(Fn("is_even"));    // <- the above is equivalent to this

array.filter(|n| n % 2 == 0);   // <- ... or this
```

Built-in Functions
------------------

The following standard methods (mostly defined in the [`BasicFnPackage`][built-in packages] but
excluded when using a [raw `Engine`]) operate on function pointers.

| Function                           | Parameter(s) | Description                                                                                      |
| ---------------------------------- | ------------ | ------------------------------------------------------------------------------------------------ |
| `name` method and property         | _none_       | returns the name of the [function] encapsulated by the function pointer                          |
| `is_anonymous` method and property | _none_       | does the function pointer refer to an [anonymous function]? Not available under [`no_function`]. |
| `call`                             | _arguments_  | calls the [function] matching the function pointer's name with the _arguments_                   |


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


```admonish warning "Not First-Class Functions"

Beware that function pointers are _not_ first-class functions.

They are _syntactic sugar_ only, capturing only the _name_ of a [function] to call.
They do not hold the actual [functions].

The actual [function] must be defined in the appropriate [namespace][function namespace]
for the call to succeed.
```

~~~admonish warning "Global Namespace Only"

Because of their dynamic nature, function pointers cannot refer to functions in [`import`]-ed [modules].

They can only refer to [functions] within the global [namespace][function namespace].

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
~~~


Dynamic Dispatch
----------------

The purpose of function pointers is to enable rudimentary _dynamic dispatch_, meaning to determine,
at runtime, which function to call among a group.

Although it is possible to simulate dynamic dispatch via a number and a large
[`if-then-else-if`][`if`] statement, using function pointers significantly simplifies the code.

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

Therefore, obviously, binding the `this` pointer is unsupported under [`no_object`].


Call a Function Pointer within a Rust Function (as a Callback)
--------------------------------------------------------------

It is completely normal to register a Rust function with an [`Engine`] that takes parameters
whose types are function pointers.  The Rust type in question is `rhai::FnPtr`.

A function pointer in Rhai is essentially syntactic sugar wrapping the _name_ of a function
to call in script.  Therefore, the script's _execution context_ (i.e. [`NativeCallContext`])
is needed in order to call a function pointer.

```rust
use rhai::{Engine, FnPtr, NativeCallContext};

let mut engine = Engine::new();

// A function expecting a callback in form of a function pointer.
fn super_call(context: NativeCallContext, callback: FnPtr, value: i64)
                -> Result<String, Box<EvalAltResult>>
{
    // Use 'FnPtr::call_within_context' to call the function pointer using the call context.
    // 'FnPtr::call_within_context' automatically casts to the required result type.
    callback.call_within_context(&context, (value,))
    //                                     ^^^^^^^^ arguments passed in tuple
}

engine.register_fn("super_call", super_call);
```


Call a Function Pointer Directly
--------------------------------

The `FnPtr::call` method allows the function pointer to be called directly on any [`Engine`] and
[`AST`], making it possible to reuse the `FnPtr` data type in may different calls and scripting
environments.

```rust
use rhai::{Engine, FnPtr};

let engine = Engine::new();

// Compile script to AST
let ast = engine.compile(
r#"
    let test = "hello";
    |x| test + x            // this creates a closure
"#)?;

// Save the closure together with captured variables
let fn_ptr = engine.eval_ast::<FnPtr>(&ast)?;

// 'f' captures: the Engine, the AST, and the closure
let f = move |x: i64| -> Result<String, _> {
            fn_ptr.call(&engine, &ast, (x,))
        };

// 'f' can be called like a normal function
let result = f(42)?;

result == "hello42";
```


Bind to a native Rust Function
------------------------------

It is also possible to create a function pointer that binds to a native Rust function or a Rust closure.

The signature of the native Rust function takes the following form.

> ```rust
> Fn(context: NativeCallContext, args: &mut [&mut Dynamic])  
>    -> Result<Dynamic, Box<EvalAltResult>> + 'static
> ```

where:

| Parameter |         Type          | Description                                     |
| --------- | :-------------------: | ----------------------------------------------- |
| `context` | [`NativeCallContext`] | mutable reference to the current _call context_ |
| `args`    | `&mut [&mut Dynamic]` | mutable reference to list of arguments          |

When such a function pointer is used in script, the native Rust function will be called
with the arguments provided.

The Rust function should check whether the appropriate number of arguments have been passed.
