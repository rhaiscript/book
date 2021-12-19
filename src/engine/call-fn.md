Call Rhai Functions from Rust
============================

{{#include ../links.md}}

Rhai also allows working _backwards_ from the other direction &ndash; i.e. calling a Rhai-scripted
[function] from Rust via `Engine::call_fn`.

```rust no_run
┌─────────────┐
│ Rhai script │
└─────────────┘

import "process" as proc;           // this is evaluated every time

fn hello(x, y) {
    // hopefully 'my_var' is in scope when this is called
    x.len + y + my_var
}

fn hello(x) {
    // hopefully 'my_string' is in scope when this is called
    x * my_string.len()
}

fn hello() {
    // hopefully 'MY_CONST' is in scope when this is called
    if MY_CONST {
        proc::process_data(42);     // can access imported module
    }
}

┌──────┐
│ Rust │
└──────┘

// Compile the script to AST
let ast = engine.compile(script)?;

// Create a custom 'Scope'
let mut scope = Scope::new();

// A custom 'Scope' can also contain any variables/constants available to
// the functions
scope.push("my_var", 42_i64);
scope.push("my_string", "hello, world!");
scope.push_constant("MY_CONST", true);

// Evaluate a function defined in the script, passing arguments into the
// script as a tuple.
//
// Beware, arguments must be of the correct types because Rhai does not
// have built-in type conversions. If arguments of the wrong types are passed,
// the Engine will not find the function.
//
// Variables/constants pushed into the custom 'Scope'
// (i.e. 'my_var', 'my_string', 'MY_CONST') are visible to the function.

let result: i64 = engine.call_fn(&mut scope, &ast, "hello", ( "abc", 123_i64 ) )?;
//          ^^^                                             ^^^^^^^^^^^^^^^^^^
//          return type must be specified                   put arguments in a tuple

let result: i64 = engine.call_fn(&mut scope, &ast, "hello", ( 123_i64, ) )?;
//                                                          ^^^^^^^^^^^^ tuple of one

let result: i64 = engine.call_fn(&mut scope, &ast, "hello", () )?;
//                                                          ^^ unit = tuple of zero
```

When using `Engine::call_fn`, the [`AST`] is first evaluated before the [function] is called.
This is usually desirable in order to [import][`import`] the necessary external [modules] that are
needed by the [function].

All new [variables]/[constants] introduced are, by default, _not_ retained inside the [`Scope`].
In other words, the [`Scope`] is _rewound_ to the initial size after each call.

If these default behaviors are not desirable, use `Engine::call_fn_raw`.


`FuncArgs` trait
----------------

`Engine::call_fn` takes a parameter of any type that implements the [`FuncArgs`][traits] trait,
which is used to parse a data type into individual argument values for the [function] call.

Rhai implements [`FuncArgs`][traits] for tuples and `Vec<T>`.

Custom types (e.g. structures) can also implement [`FuncArgs`][traits] so they can be used for
calling `Engine::call_fn`.

```rust no_run
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


Low-Level API &ndash; `Engine::call_fn_raw`
------------------------------------------

For more control, construct all arguments as `Dynamic` values and use `Engine::call_fn_raw`,
passing it anything that implements `AsMut<[Dynamic]>` (such as a simple array or a `Vec<Dynamic>`):

```rust no_run
let result = engine.call_fn_raw(
                &mut scope,         // scope to use
                &ast,               // AST containing the functions
                false,              // false = do not evaluate the AST
                false,              // false = do not rewind the scope (i.e. keep new variables)
                "hello",            // function entry-point
                None,               // 'this' pointer, if any
                [ "abc".into(), 123_i64.into() ]    // arguments
             )?;
```

`Engine::call_fn_raw` extends control to the following:

* Whether to skip evaluation of the [`AST`] before calling the target [function]
* Whether to rewind the custom [`Scope`] at the end of the [function] call
* Whether to bind the `this` pointer to a specific value

### Skip evaluation of the `AST`

By default, the [`AST`] is evaluated before calling the target [function].
A parameter can be passed to skip this evaluation.

### Keep new variables/constants

By default, the [`Engine`] _rewinds_ the custom [`Scope`] after each call to the initial size,
so any new [variable]/[constant] defined are cleared and will not spill into the custom [`Scope`].

This keeps the [`Scope`] from being continuously polluted by new [variables] and is usually the
expected intuitive behavior.

A parameter can be passed to keep new [variables]/[constants] within the custom [`Scope`].
This allows the [function] to easily pass values back to the caller by leaving them inside the
custom [`Scope`].

When doing so, however, beware that all [variables]/[constants] defined at top level of the
[function] or in the script body will _persist_ inside the custom [`Scope`].  If any of them are
temporary and not intended to be retained, define them inside a statements block.

```rust no_run
┌─────────────┐
│ Rhai script │
└─────────────┘

fn initialize() {
    let x = 42;                     // 'x' is retained
    let y = x * 2;                  // 'y' is retained

    // Use a new statements block to define temp variables
    {
        let temp = x + y;           // 'temp' is NOT retained

        foo = temp * temp;          // 'foo' is visible in the scope
    }
}

let foo = 123;                      // 'foo' is retained

// Use a new statements block to define temp variables
{
    let bar = foo / 2;              // 'bar' is NOT retained

    foo = bar * bar;
}

┌──────┐
│ Rust │
└──────┘

engine.call_fn_raw(&mut scope, &ast, true, false, "initialize", None, [])?;
//                                   ^^^^ evaluate AST before call
//                                         ^^^^^ do not rewind scope

// At this point, 'scope' contains these variables: 'foo', 'x', 'y'
```

### Bind the `this` pointer

`Engine::call_fn_raw` can also bind a value to the `this` pointer of a script-defined [function].

This functionality is not available to `Engine::call_fn` which cannot call functions in
_method-call_ style.

```rust no_run
let ast = engine.compile("fn action(x) { this += x; }")?;

let mut value: Dynamic = 1_i64.into();

engine.call_fn_raw(
            &mut scope,
            &ast,
            false,
            false,
            "action",
            Some(&mut value),       // binding the 'this' pointer
            [ 41_i64.into() ]
       )?;

assert_eq!(value.as_int()?, 42);
```
