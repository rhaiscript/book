Register a Rust Function for Use in Rhai Scripts
================================================

{{#include ../links.md}}

Rhai's scripting engine is very lightweight.  It gets most of its abilities from functions.

To call these functions, they need to be _registered_ via `Engine::register_fn`.

```admonish tip.small "Tip: Function overloading"

Functions registered with the [`Engine`] can be _overloaded_ as long as the _signature_ is unique,
i.e. different functions can have the same name as long as their parameters are of different types
or different numbers (i.e. _arity_).

New definitions _overwrite_ previous definitions of the same name, same arity and same parameter types.
```

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

engine.register_fn("add", add_len)
      .register_fn("add", add_len_count)
      .register_fn("add", get_any_value)
      .register_fn("inc", |x: i64| {    // closure is also OK!
          x + 1
      })
      .register_fn("log", |label: &str, x: i64| {
          println!("{} = {}", label, x);
      });

let result = engine.eval::<i64>(r#"add(40, "xx")"#)?;

println!("Answer: {}", result);         // prints 42

let result = engine.eval::<i64>(r#"add(40, "x", 2)"#)?;

println!("Answer: {}", result);         // prints 42

let result = engine.eval::<i64>("add()")?;

println!("Answer: {}", result);         // prints 42

let result = engine.eval::<i64>("inc(41)")?;

println!("Answer: {}", result);         // prints 42

engine.run(r#"log("value", 42)"#)?;     // prints "value = 42"
```

~~~admonish tip.small "Tip: Use closures"

It is common for short functions to be registered via a _closure_.

```rust
engine.register_fn("foo", |x: i64, y: bool| ...);
```
~~~

~~~admonish tip.small "Tip: Create a `Dynamic`"

To create a [`Dynamic`] value, use `Dynamic::from`.

[Standard types] in Rhai can also use `.into()`.

```rust
use rhai::Dynamic;

let obj = TestStruct::new();

let x = Dynamic::from(obj);

// '.into()' works for standard types

let x = 42_i64.into();

let y = "hello!".into();
```
~~~
