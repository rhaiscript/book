Call Method as Function
=======================

{{#include ../links.md}}


Method-Call Style vs. Function-Call Style
-----------------------------------------

### Method-call syntax

> _object_ `.` _function_ `(` _parameter_`,` ... `,` _parameter_`)`

~~~admonish warning.small "_Method-call_ style not supported under [`no_object`]"
```rust
// Below is a syntax error under 'no_object'.
engine.run("let x = [42]; x.clear();")?;
                        // ^ cannot call method-style
```
~~~

### Function-call syntax

> _function_ `(` _object_`,` _parameter_`,` ... `,` _parameter_`)`

### Equivalence

```admonish note.side

This design is similar to Rust.
```

Internally, methods on a [custom type] is _the same_ as a function taking a `&mut` first argument of
the object's type.

Therefore, methods and functions can be called interchangeably.

```rust
impl TestStruct {
    fn foo(&mut self) -> i64 {
        self.field
    }
}

engine.register_fn("foo", TestStruct::foo);

let result = engine.eval::<i64>(
"
    let x = new_ts();
    foo(x);                         // normal call to 'foo'
    x.foo()                         // 'foo' can also be called like a method on 'x'
")?;

println!("result: {}", result);     // prints 1
```


First `&mut` Parameter
----------------------

The opposite direction also works &mdash; methods in a Rust [custom type] registered with the
[`Engine`] can be called just like a regular function.  In fact, like Rust, object methods are
registered as regular [functions] in Rhai that take a first `&mut` parameter.

Unlike functions defined in script (for which all arguments are passed by _value_),
native Rust functions may mutate the first `&mut` argument.

Sometimes, however, there are more subtle differences. Methods called in normal function-call style
may end up not muting the object afterall &mdash; see the example below.

Custom types, [properties][getters/setters], [indexers] and methods are disabled under the
[`no_object`] feature.

```rust
let a = new_ts();   // constructor function
a.field = 500;      // property setter
a.update();         // method call, 'a' can be modified

update(a);          // <- this de-sugars to 'a.update()'
                    //    'a' can be modified and is not a copy

let array = [ a ];

update(array[0]);   // <- 'array[0]' is an expression returning a calculated value,
                    //    a transient (i.e. a copy), so this statement has no effect
                    //    except waste time cloning 'a'

array[0].update();  // <- call in method-call style will update 'a'
```

```admonish danger.small "No support for references"

Rhai does NOT support normal references (i.e. `&T`) as parameters.
All references must be mutable (i.e. `&mut T`).
```


Number of Parameters in Methods
-------------------------------

Native Rust methods registered with an [`Engine`] take _one additional parameter_ more than
an equivalent method coded in script, where the object is accessed via the `this` pointer instead.

The following table illustrates the differences:

| Function type | No. of parameters |     Object reference     |      Function signature       |
| :-----------: | :---------------: | :----------------------: | :---------------------------: |
|  Native Rust  |      _N_ + 1      | first `&mut T` parameter | `Fn(obj: &mut T, x: U, y: V)` |
|  Rhai script  |        _N_        |          `this`          |       `Fn(x: U, y: V)`        |


`&mut` is Efficient, Except for `&mut ImmutableString`
------------------------------------------------------

Using a `&mut` first parameter is highly encouraged when using types that are expensive to clone,
even when the intention is not to mutate that argument, because it avoids cloning that argument value.

Even when a function is never intended to be a method &ndash; for example an operator,
it is still sometimes beneficial to make it method-like (i.e. with a first `&mut` parameter)
if the first parameter is not modified.

For types that are expensive to clone (remember, all function calls are passed cloned
copies of argument values), this may result in a significant performance boost.

For primary types that are cheap to clone (e.g. those that implement `Copy`), including `ImmutableString`,
this is not necessary.

```rust
// This is a type that is very expensive to clone.
#[derive(Debug, Clone)]
struct VeryComplexType { ... }

// Calculate some value by adding 'VeryComplexType' with an integer number.
fn do_add(obj: &VeryComplexType, offset: i64) -> i64 {
    ...
}

engine.register_type::<VeryComplexType>()
      .register_fn("+", add_pure /* or  add_method*/);

// Very expensive to call, as the 'VeryComplexType' is cloned before each call.
fn add_pure(obj: VeryComplexType, offset: i64) -> i64 {
    do_add(obj, offset)
}

// Efficient to call, as only a reference to the 'VeryComplexType' is passed.
fn add_method(obj: &mut VeryComplexType, offset: i64) -> i64 {
    do_add(obj, offset)
}
```


Data Race Considerations
------------------------

```admonish note.side.wide "Data races"

Data races are not possible in Rhai under the [`no_closure`] feature because no sharing ever occurs.
```

Because methods always take a mutable reference as the first argument, even it the value is never changed,
care must be taken when using _shared_ values with methods.

Usually data races are not possible in Rhai because, for each function call, there is ever only one
value that is mutable &ndash; the first argument of a method.  All other arguments are cloned.

It is possible, however, to create a data race with a _shared_ value, when the same value is
_captured_ in a [closure] and then used again as the _object_ of calling that [closure]!

```rust
let x = 20;

x.is_shared() == false;             // 'x' is not shared, so no data race is possible

let f = |a| this += x + a;          // 'x' is captured in this closure

x.is_shared() == true;              // now 'x' is shared

x.call(f, 2);                       // <- error: data race detected on 'x'
```
