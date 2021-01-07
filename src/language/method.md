Call Method as Function
======================

{{#include ../links.md}}


First `&mut` Parameter
----------------------

Property [getters/setters] and [methods][custom types] in a Rust custom type registered with the [`Engine`] can be called
just like a regular function.  In fact, like Rust, property getters/setters and object methods
are registered as regular [functions] in Rhai that take a first `&mut` parameter.

Unlike functions defined in script (for which all arguments are passed by _value_),
native Rust functions may mutate the object (or the first argument if called in normal function call style).

However, sometimes it is not as straight-forward, and methods called in function-call style may end up
not muting the object &ndash; see the example below. Therefore, it is best to always use method-call style.

Custom types, properties and methods can be disabled via the [`no_object`] feature.

```rust
let a = new_ts();   // constructor function
a.field = 500;      // property setter
a.update();         // method call, 'a' can be modified

update(a);          // <- this de-sugars to 'a.update()' thus if 'a' is a simple variable
                    //    unlike scripted functions, 'a' can be modified and is not a copy

let array = [ a ];

update(array[0]);   // <- 'array[0]' is an expression returning a calculated value,
                    //    a transient (i.e. a copy), so this statement has no effect
                    //    except waste a lot of time cloning

array[0].update();  // <- call in method-call style will update 'a'
```

**IMPORTANT: Rhai does NOT support normal references (i.e. `&T`) as parameters.**


Number of Parameters in Methods
------------------------------

Native Rust methods registered with an [`Engine`] take _one additional parameter_ more than
an equivalent method coded in script, where the object is accessed via the `this` pointer instead.

The following table illustrates the differences:

| Function type | Parameters |     Object reference      |      Function signature       |
| :-----------: | :--------: | :-----------------------: | :---------------------------: |
|  Native Rust  |  _N_ + 1   | first `&mut T` parameter  | `Fn(obj: &mut T, x: U, y: V)` |
|  Rhai script  |    _N_     | `this` (of type `&mut T`) |       `Fn(x: U, y: V)`        |


`&mut` is Efficient, Except for `&mut ImmutableString`
----------------------------------------------------

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

engine
    .register_type::<VeryComplexType>()
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
