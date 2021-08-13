Constants
=========

{{#include ../links.md}}

Constants can be defined using the `const` keyword and are immutable.

Constants follow the same naming rules as [variables], but as a convention are often named with
all-capital letters.

```rust no_run
const X = 42;

print(X * 2);       // prints 84

X = 123;            // <- syntax error: constant modified
```

```rust no_run
const X;            // 'X' is a constant '()'

const X = 40 + 2;   // 'X' is a constant 42
```


Manually Add Constant into Custom Scope
--------------------------------------

It is possible to add a constant into a custom [`Scope`] so it'll be available to scripts
running with that [`Scope`].

It is very useful to have a constant value hold a [custom type], which essentially acts
as a [_singleton_](../patterns/singleton.md).

```rust no_run
use rhai::{Engine, Scope};

#[derive(Debug, Clone)]
struct TestStruct(i64);                                     // custom type

let mut engine = Engine::new();

engine
    .register_type_with_name::<TestStruct>("TestStruct")    // register custom type
    .register_get("value", |obj: &mut TestStruct| obj.0),   // property getter
    .register_fn("update_value",
        |obj: &mut TestStruct, value: i64| obj.0 = value    // mutating method
    );

let script =
"
    MY_NUMBER.update_value(42);
    print(MY_NUMBER.value);
";

let ast = engine.compile(script)?;

let mut scope = Scope::new();                               // create custom scope

scope.push_constant("MY_NUMBER", TestStruct(123_i64));      // add constant variable

// Beware: constant objects can still be modified via a method call!
engine.run_ast_with_scope(&mut scope, &ast)?;               // prints 42

// Running the script directly, as below, is less desirable because
// the constant 'MY_NUMBER' will be propagated and copied into each usage
// during the script optimization step
engine.run_with_scope(&mut scope, script)?;
```


Caveat &ndash; Constants Can be Modified via Rust
------------------------------------------------

A custom type stored as a constant cannot be modified via script, but _can_ be modified via
a registered Rust function that takes a first `&mut` parameter &ndash; because there is no way for
Rhai to know whether the Rust function modifies its argument!

By default, native Rust functions with a first `&mut` parameter always allow constants to be passed
to them. This is because using `&mut` can avoid unnecessary cloning of a [custom type] value, even
though it is actually not modified &ndash; for example returning the size of a collection type.

In line with intuition, Rhai is smart enough to always pass a _cloned copy_ of a constant as the
first `&mut` argument if the function is called in normal function call style.

If it is called as a [method], however, the Rust function will be able to modify the constant's value.

Also, property [setters][getters/setters] and [indexers] are always assumed to mutate the first
`&mut` parameter and so they always raise errors when passed constants by default.

```rust no_run
// For the below, assume 'increment' is a Rust function with '&mut' first parameter

const X = 42;       // a constant

increment(X);       // call 'increment' in normal FUNCTION-CALL style
                    // since 'X' is constant, a COPY is passed instead

X == 42;            // value is 'X" is unchanged

X.increment();      // call 'increment' in METHOD-CALL style

X == 43;            // value of 'X' is changed!
                    // must use 'Dynamic::is_read_only' to check if parameter is constant

fn double() {
    this *= 2;      // function doubles 'this'
}

let y = 1;          // 'y' is not constant and mutable

y.double();         // double it...

y == 2;             // value of 'y' is changed as expected

X.double();         // since 'X' is constant, a COPY is passed to 'this'

X == 43;            // value of 'X' is unchanged by script
```

### Pure plugin functions

When functions are registered as part of a [plugin], `&mut` parameters are protected from being
passed constant values.

By default, [plugin functions] that take a first `&mut` parameter disallow passing constants as that
parameter.

However, if a [plugin function] is marked with the `#[export_fn(pure)]` or `#[rhai_fn(pure)]` attribute,
it is considered _pure_ (i.e. assumed to not modify its arguments) and so constants are allowed.

### Implications on script optimization

Rhai _assumes_ that constants are never changed, even via Rust functions.

This is important to keep in mind because the script [optimizer][script optimization]
by default does _constant propagation_ as a operation.

If a constant is eventually modified by a Rust function, the optimizer will not see
the updated value and will propagate the original initialization value instead.

`Dynamic::is_read_only` can be used to detect whether a [`Dynamic`] value is constant or not
within a Rust function.
