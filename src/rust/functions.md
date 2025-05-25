Register a Rust Function for Use in Rhai Scripts
================================================

{{#include ../links.md}}

Rhai's scripting engine is very lightweight.  It gets most of its abilities from functions.

To call these functions, they need to be _registered_ via `Engine::register_fn`.

```rust
use rhai::{Dynamic, Engine, ImmutableString};

// Normal function that returns a standard type
// Remember to use 'ImmutableString' and not 'String'
fn add_len(x: i64, s: ImmutableString) -> i64 {
    x + s.len()
}
// Alternatively, '&str' maps directly to 'ImmutableString'
fn add_len_count(x: i64, s: &str, c: i64) -> i64 {
    x + s.len() * c
}
// Function that returns a 'Dynamic' value
fn get_any_value() -> Dynamic {
    42_i64.into()                       // standard types can use '.into()'
}

let mut engine = Engine::new();

// Notice that all three functions are overloaded into the same name with
// different number of parameters and/or parameter types.
engine.register_fn("add", add_len)
      .register_fn("add", add_len_count)
      .register_fn("add", get_any_value)
      .register_fn("inc", |x: i64| {    // closure is also OK!
          x + 1
      })
      .register_fn("log", |label: &str, x: i64| {
          println!("{label} = {x}");
      });

let result = engine.eval::<i64>(r#"add(40, "xx")"#)?;

println!("Answer: {result}");           // prints 42

let result = engine.eval::<i64>(r#"add(40, "x", 2)"#)?;

println!("Answer: {result}");           // prints 42

let result = engine.eval::<i64>("add()")?;

println!("Answer: {result}");           // prints 42

let result = engine.eval::<i64>("inc(41)")?;

println!("Answer: {result}");           // prints 42

engine.run(r#"log("value", 42)"#)?;     // prints "value = 42"
```

~~~admonish tip "Tip: Use closures"

It is common for short functions to be registered via a _closure_.

```rust
┌──────┐
│ Rust │
└──────┘

engine.register_fn("foo", |x: i64, y: i64| x * 2 + y * 3);
//                            ^^^     ^^^
// Usually parameter types need to be specified

engine.register_fn("bar", |x: i64| -> Result<_, Box<EvalAltResult>> { x * 2 });
//                                    ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
// For fallible closures, the return ERROR type may need to be specified

┌─────────────┐
│ Rhai script │
└─────────────┘

foo(42, 100);       // <- 42 * 2 + 100 * 3
```
#### Interact with external environment

An additional benefit to using closures is that they can capture external variables.

For example, capturing a type wrapped in shared mutability (e.g. `Rc<RefCell<T>>`)
allows a script to interact with the external environment through that shared type.

See also: [Control Layer]({{rootUrl}}/patterns/control.md).

```rust
┌──────┐
│ Rust │
└──────┘

/// A type that encapsulates some behavior.
#[derive(Clone)]
struct TestStruct { ... }

impl TestStruct {
    /// Some action defined on that type.
    pub fn do_foo(&self, x: i64, y: bool) {
        // ... do something drastic with x and y
    }
}

/// Wrapped in shared mutability: Rc<RefCell<TestStruct>>.
let shared_obj = Rc::new(RefCell::new(TestStruct::new()));

/// Clone the shared reference and move it into the closure.
let embedded_obj = shared.clone();

engine.register_fn("foo", move |x: i64, y: bool| {
//                        ^^^^ 'embedded_obj' is captured into the closure

    embedded_obj.borrow().do_foo(x, y);
});

┌─────────────┐
│ Rhai script │
└─────────────┘

foo(42, true);      // <- equivalent to: shared_obj.borrow().do_foo(42, true);
```
~~~

~~~admonish warning "Warning: Don't touch those generic parameters"

Rhai uses an intricate system of traits (in particular `RhaiNativeFunc`) with many generic parameters to ensure
an intuitive and smooth developer experience that _just works_.

Because of this, it is not recommended to touch those generic parameters directly.  These generic parameters may change
liberally in future versions of Rhai. In most situations they are automatically inferred by the compiler.

In the cases where the compiler fail to infer types when registering a _closure_, (usually with the _error_ type
of a [fallible function]), manually declare the parameter and/or return types.

```rust
// The following fails to compile because the compiler does not know
// the return _error_ type of the closure.
// It knows the return type, which is 'Result<i64, E>', but 'E' is not known.
engine.register_fn("foo", |x: i64| Ok(x));

// Don't do this...
engine.register_fn::<_, 1, false, i64, Box<EvalAltResult>>("foo", |x: i64| Ok(x));

// Do this...
engine.register_fn("foo", |x: i64| -> Result<i64, Box<EvalAltResult>> { Ok(x) });
```
~~~
