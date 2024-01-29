Manually Register Custom Type
=============================

{{#include ../links.md}}


```admonish warning.small "Warning"

This assumes that the type is defined in an external crate and so the [`CustomType`] trait
cannot be implemented for it due to Rust's [_orphan rule_](https://doc.rust-lang.org/book/ch10-02-traits.html).
```

```admonish tip.side "Tip: Working with enums"

It is also possible to use Rust enums with Rhai.

See the pattern [Working with Enums]({{rootUrl}}/patterns/enums.md) for more details.
```

The custom type needs to be _registered_ into an [`Engine`] via:

| `Engine` API                   | `type_of` output    |
| ------------------------------ | ------------------- |
| `register_type::<T>`           | full Rust path name |
| `register_type_with_name::<T>` | friendly name       |

```rust
use rhai::{Engine, EvalAltResult};

#[derive(Debug, Clone)]
struct TestStruct {
    field: i64
}

impl TestStruct {
    fn new() -> Self {
        Self { field: 1 }
    }
}

let mut engine = Engine::new();

// Register custom type with friendly name
engine.register_type_with_name::<TestStruct>("TestStruct")
      .register_fn("new_ts", TestStruct::new);

// Cast result back to custom type.
let result = engine.eval::<TestStruct>(
"
    new_ts()        // calls 'TestStruct::new'
")?;

println!("result: {}", result.field);   // prints 1

```

`type_of()` a Custom Type
-------------------------

```admonish question.side.wide "Giving types the same name?"

It is OK to register several custom types under the _same_ friendly name
and `type_of()` will faithfully return it.

How this might possibly be useful is left to the imagination of the user.
```

[`type_of()`] works fine with custom types and returns the name of the type.

If `Engine::register_type_with_name` is used to register the custom type with a special
"pretty-print" friendly name, [`type_of()`] will return that name instead.

```rust
engine.register_type::<TestStruct1>()
      .register_fn("new_ts1", TestStruct1::new)
      .register_type_with_name::<TestStruct2>("TestStruct")
      .register_fn("new_ts2", TestStruct2::new);

let ts1_type = engine.eval::<String>("let x = new_ts1(); x.type_of()")?;
let ts2_type = engine.eval::<String>("let x = new_ts2(); x.type_of()")?;

println!("{ts1_type}");                 // prints 'path::to::TestStruct'
println!("{ts2_type}");                 // prints 'TestStruct'
```


`==` Operator
-------------

Many standard functions (e.g. filtering, searching and sorting) expect a custom type to be
_comparable_, meaning that the `==` operator must be registered for the custom type.

For example, in order to use the [`in`] operator with a custom type for an [array],
the `==` operator is used to check whether two values are the same.

```rust
// Assume 'TestStruct' implements `PartialEq`
engine.register_fn("==",
    |item1: &mut TestStruct, item2: TestStruct| item1 == &item2
);

// Then this works in Rhai:
let item = new_ts();        // construct a new 'TestStruct'
item in array;              // 'in' operator uses '=='
```
