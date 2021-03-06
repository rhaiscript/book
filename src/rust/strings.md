`String` Parameters in Rust Functions
====================================

{{#include ../links.md}}


Avoid `String`
--------------

As much as possible, avoid using `String` parameters in functions.

Each `String` argument is cloned during _every_ single call to that function &ndash;
and the copy immediately thrown away right after the call.

Needless to say, it is _extremely_ inefficient to use `String` parameters.


`&str` Maps to `ImmutableString`
-------------------------------

Rust functions accepting parameters of `String` should use `&str` instead because it maps directly
to [`ImmutableString`] which is the type that Rhai uses to represent [strings] internally.

The parameter type `String` involves always converting an [`ImmutableString`] into a `String`
which mandates cloning it.

Using `ImmutableString` or `&str` is much more efficient.

A common mistake made by novice Rhai users is to register functions with `String` parameters.

```rust,no_run
fn get_len1(s: String) -> i64 {             // very inefficient!!!
    s.len() as i64
}
fn get_len2(s: &str) -> i64 {               // this is better
    s.len() as i64
}
fn get_len3(s: ImmutableString) -> i64 {    // the above is equivalent to this
    s.len() as i64
}

engine
    .register_fn("len1", get_len1)
    .register_fn("len2", get_len2)
    .register_fn("len3", get_len3);

let len = engine.eval::<i64>("len1(x)")?;   // 'x' is cloned, very inefficient!
let len = engine.eval::<i64>("len2(x)")?;   // 'x' is shared
let len = engine.eval::<i64>("len3(x)")?;   // 'x' is shared
```


`&mut String` Does Not Work As Expected
--------------------------------------

A function with the first parameter being `&mut String` does not match a string argument passed to it,
which has type `ImmutableString`.

In fact, `&mut String` is treated as an opaque [custom type].

```rust,no_run
fn bad(s: &mut String) { ... }              // '&mut String' will not match string values

fn good(s: &mut ImmutableString) { ... }

engine
    .register_fn("bad", bad)
    .register_fn("good", good);

engine.eval(r#"bad("hello")"#)?;            // <- error: function 'bad (string)' not found

engine.eval(r#"good("hello")"#)?;           // <- this one works
```
