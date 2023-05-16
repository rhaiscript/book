`Dynamic` Return Value
======================

{{#include ../links.md}}

Rhai supports registering functions that return [`Dynamic`].

A [`Dynamic`] value can hold any clonable type.

```rust
use rhai::{Engine, Dynamic};

// The 'Dynamic' return type allows this function to
// return values of any supported type!
fn get_info(bag: &mut PropertyBag, key: &str) -> Dynamic {
    if let Some(prop_type) = bag.get_type(key) {
        match prop_type {
            // Use '.into()' for standard types
            "string" => bag.get::<&str>(key).into(),
            "int" => bag.get::<i64>(key).into(),
            "bool" => bag.get::<bool>(key).into(),
                            :
                            :
            // Use 'Dynamic::from' for custom types
            "bag" => Dynamic::from(bag.get::<PropertyBag>(key))
        }
    } else {
        // Return () upon error
        Dynamic::UNIT
    }
}

let mut engine = Engine::new();

engine.register_fn("get_info", get_info);
```

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


Alternative to Fallible Functions
---------------------------------

Instead of registering a [fallible function], it is usually more idiomatic to leverage the _dynamic_
nature of Rhai and simply return [`()`] upon error.

```rust
use rhai::{Engine, Dynamic};

// Function that may fail - return () upon failure
fn safe_divide(x: i64, y: i64) -> Dynamic {
    if y == 0 {
        // Return () to indicate an error if y is zero
        Dynamic::UNIT
    } else {
        // Use '.into()' to convert standard types to 'Dynamic'
        (x / y).into()
    }
}

let mut engine = Engine::new();

engine.register_fn("divide", safe_divide);

// The following prints 'error!'
engine.run(r#"
    let result = divide(40, 0);
    
    if result == () {
        print("error!");
    } else {
        print(result);
    }
"#)?;
```
