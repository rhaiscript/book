Function Pointers
=================

{{#include ../links.md}}

```admonish question.side.wide "Trivia"

A function pointer simply stores the _name_ of the [function] as a [string].
```

It is possible to store a _function pointer_ in a variable just like a normal value.

A function pointer is created via the `Fn` [function], which takes a [string] parameter.

Call a function pointer via the `call` method.


Built-in Functions
------------------

The following standard methods (mostly defined in the [`BasicFnPackage`][built-in packages] but excluded if
using a [raw `Engine`]) operate on function pointers.

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


Warning &ndash; Not First-Class Functions
-----------------------------------------

Beware that function pointers are _not_ first-class functions.

They are _syntactic sugar_ only, capturing only the _name_ of a [function] to call.
They do not hold the actual [functions].

The actual [function] must be defined in the appropriate [namespace][function namespace]
for the call to succeed.

~~~admonish bug "Cannot export function pointer"

[Exporting][`export`] a function pointer (or an [anonymous function] or [closure])
from a [module] referring to a local [function] fails at runtime.

That is because the target [function] is not supposed to be found in the caller's
[namespace][function namespace].

```js
┌────────────────┐
│ my_module.rhai │
└────────────────┘

fn increment(x) {
    x + 1
}

export let inc = Fn("increment");   // exports a function pointer


┌───────────┐
│ main.rhai │
└───────────┘

import "my_module" as my_mod;

print(my_mod::increment(41));       // ok!

let x = my_mod::inc.call(41);       // runtime error:
                                    //    function 'increment' not found
```
~~~


Warning &ndash; Global Namespace Only
-------------------------------------

```admonish info.side "See also"

See _[Function Namespaces]_ for more details.
```

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
    Fn("method1")
} else if x == 0 {
    Fn("method2")
} else if x > 0 {
    Fn("method3")
};

// Dynamic dispatch
func.call(42);

// Using functions map
let map = [ Fn("method1"), Fn("method2"), Fn("method3") ];

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

let func = Fn("add");       // function pointer to 'add'

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

engine.register_result_fn("super_call", super_call);
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
