Register a Rust Function
========================

{{#include ../links.md}}

Rhai's scripting engine is very lightweight.  It gets most of its abilities from functions.

To call these functions, they need to be _registered_ via `Engine::register_fn` and
`Engine::register_result_fn` (see [fallible functions]).

```rust no_run
use rhai::{Dynamic, Engine, EvalAltResult, ImmutableString};

// Normal function that returns a standard type
// Remember to use 'ImmutableString' and not 'String'
fn add_len(x: i64, s: ImmutableString) -> i64 {
    x + s.len()
}
// Alternatively, '&str' maps directly to 'ImmutableString'
fn add_len_str(x: i64, s: &str) -> i64 {
    x + s.len()
}
// Function that returns a 'Dynamic' value
fn get_any_value() -> Dynamic {
    42_i64.into()                       // standard types can use '.into()'
}

let mut engine = Engine::new();

engine.register_fn("add", add_len)
      .register_fn("add_str", add_len_str)
      .register_fn("get_any_value", get_any_value);

let result = engine.eval::<i64>(r#"add(40, "xx")"#)?;

println!("Answer: {}", result);         // prints 42

let result = engine.eval::<i64>(r#"add_str(40, "xx")"#)?;

println!("Answer: {}", result);         // prints 42

let result = engine.eval::<i64>("get_any_value()")?;

println!("Answer: {}", result);         // prints 42
```

To create a [`Dynamic`] value, use the `Dynamic::from` method.
[Standard types] in Rhai can also use `.into()`.

```rust no_run
use rhai::Dynamic;

let x = 42_i64.into();                  // '.into()' works for standard types

let y = "hello!".into();

let x = Dynamic::from(TestStruct::new());
```


Function Overloading
--------------------

Functions registered with the [`Engine`] can be _overloaded_ as long as the _signature_ is unique,
i.e. different functions can have the same name as long as their parameters are of different types
or different number.

New definitions _overwrite_ previous definitions of the same name and same number/types of parameters.
