Methods
=======

{{#include ../links.md}}


Methods of [custom types] are registered via `Engine::register_fn`.

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

    fn update(&mut self, x: i64) {      // methods take &mut as first parameter
        self.field += x;
    }
}

let mut engine = Engine::new();

// Most Engine API's can be chained up.
engine.register_type_with_name::<TestStruct>("TestStruct")
      .register_fn("new_ts", TestStruct::new)
      .register_fn("update", TestStruct::update);

// Cast result back to custom type.
let result = engine.eval::<TestStruct>(
"
    let x = new_ts();                   // calls 'TestStruct::new'
    x.update(41);                       // calls 'TestStruct::update'
    x                                   // 'x' holds a 'TestStruct'
")?;

println!("result: {}", result.field);   // prints 42
```


First Parameter Must be `&mut`
------------------------------

_Methods_ of [custom types] take a `&mut` first parameter to that type, so that invoking methods can
always update it.

All other parameters in Rhai are passed by value (i.e. clones).

```admonish danger.small "No support for references"

Rhai does NOT support normal references (i.e. `&T`) as parameters.
All references must be mutable (i.e. `&mut T`).
```
