Call Rhai Functions from Rust
============================

{{#include ../links.md}}

Rhai also allows working _backwards_ from the other direction &ndash; i.e. calling a Rhai-scripted [function]
from Rust via `Engine::call_fn`.

[Functions] declared with `private` are hidden and cannot be called from Rust (see also [modules]).

```rust,no_run
// Define functions in a script.
let ast = engine.compile(true,
    r#"
        // a function with two parameters: string and i64
        fn hello(x, y) {
            x.len + y
        }

        // functions can be overloaded: this one takes only one parameter
        fn hello(x) {
            x * 2
        }

        // this one takes no parameters
        fn hello() {
            42
        }

        // this one is private and cannot be called by 'call_fn'
        private hidden() {
            throw "you shouldn't see me!";
        }
    "#)?;

// A custom scope can also contain any variables/constants available to the functions
let mut scope = Scope::new();

// Evaluate a function defined in the script, passing arguments into the script as a tuple.
// Beware, arguments must be of the correct types because Rhai does not have built-in type conversions.
// If arguments of the wrong types are passed, the Engine will not find the function.

let result: i64 = engine.call_fn(&mut scope, &ast, "hello", ( String::from("abc"), 123_i64 ) )?;
//          ^^^                                             ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
//        return type must be specified                          put arguments in a tuple

let result: i64 = engine.call_fn(&mut scope, &ast, "hello", (123_i64,) )?;
//                                                          ^^^^^^^^^^ tuple of one

let result: i64 = engine.call_fn(&mut scope, &ast, "hello", () )?;
//                                                          ^^ unit = tuple of zero

// The following call will return a function-not-found error because
// 'hidden' is declared with 'private'.
let result: () = engine.call_fn(&mut scope, &ast, "hidden", ())?;
```


`FuncArgs` trait
----------------

`Engine::call_fn` takes a parameter of any type that implements the [`FuncArgs`][traits] trait,
which is used to parse a data type into individual argument values for the function call.

Rhai implements [`FuncArgs`][traits] for tuples and `Vec<T>`.

Custom types (e.g. structures) can also implement [`FuncArgs`][traits] so they can be used for
calling `Engine::call_fn`.

```rust,no_run
use std::iter::once;
use rhai::FuncArgs;

// A struct containing function arguments
struct Options {
    pub foo: bool,
    pub bar: String,
    pub baz: i64
}

impl FuncArgs for Options {
    fn parse<C: Extend<Dynamic>>(self, container: &mut C) {
        container.extend(once(self.foo.into()));
        container.extend(once(self.bar.into()));
        container.extend(once(self.baz.into()));
    }
}

let options = Options { foo: true, bar: "world", baz: 42 };

// The type 'Options' can now be used as arguments to 'call_fn'!
let result: i64 = engine.call_fn(&mut scope, &ast, "hello", options)?;
```


Low-Level API &ndash; `Engine::call_fn_dynamic`
----------------------------------------------

For more control, construct all arguments as `Dynamic` values and use `Engine::call_fn_dynamic`,
passing it anything that implements `AsMut<Dynamic>` (such as a simple array or a `Vec<Dynamic>`):

```rust,no_run
let result = engine.call_fn_dynamic(
                        &mut scope,         // scope to use
                        &ast,               // AST containing the functions
                        "hello",            // function entry-point
                        None,               // 'this' pointer, if any
                        [ String::from("abc").into(), 123_i64.into() ]      // arguments
             )?;
```


### Binding the `this` Pointer

`Engine::call_fn_dynamic` can also bind a value to the `this` pointer of a script-defined function.

```rust,no_run
let ast = engine.compile("fn action(x) { this += x; }")?;

let mut value: Dynamic = 1_i64.into();

let result = engine.call_fn_dynamic(
                        &mut scope,
                        &ast,
                        "action",
                        Some(&mut value),   // binding the 'this' pointer
                        [ 41_i64.into() ]
             )?;

assert_eq!(value.as_int()?, 42);
```
