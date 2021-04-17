Custom Type Property Getters and Setters
=======================================

{{#include ../links.md}}

A [custom type] can also expose properties by registering `get` and/or `set` functions.

Properties can be accessed in a JavaScript-like syntax:

> _object_ `.` _property_
>
> _object_ `.` _property_ `=` _value_ `;`

Getters and setters each take a `&mut` reference to the first parameter.

Getters and setters are disabled when the [`no_object`] feature is used.

| `Engine` API          | Function signature(s)<br/>(`T: Clone` = custom type,<br/>`V: Clone` = data type) |        Can mutate `T`?         |
| --------------------- | -------------------------------------------------------------------------------- | :----------------------------: |
| `register_get`        | `Fn(&mut T) -> V`                                                                |      yes, but not advised      |
| `register_set`        | `Fn(&mut T, V)`                                                                  |              yes               |
| `register_get_set`    | getter: `Fn(&mut T) -> V`</br>setter: `Fn(&mut T, V)`                            | yes, but not advised in getter |
| `register_get_result` | `Fn(&mut T) -> Result<V, Box<EvalAltResult>>`                                    |      yes, but not advised      |
| `register_set_result` | `Fn(&mut T, V) -> Result<(), Box<EvalAltResult>>`                                |              yes               |

By convention, property getters are not supposed to mutate the [custom type], although there is nothing
that prevents this mutation.


Cannot Override Object Maps
--------------------------

Property getters and setters are mainly intended for [custom types].

Any getter or setter function registered for [object maps] is simply _ignored_ because
the get/set calls will be interpreted as properties on the [object maps].


Examples
--------

```rust , no_run
#[derive(Debug, Clone)]
struct TestStruct {
    field: String
}

impl TestStruct {
    // Remember &mut must be used even for getters
    fn get_field(&mut self) -> String {
        self.field.clone()
    }

    fn set_field(&mut self, new_val: &str) {
        self.field = new_val.to_string();
    }

    fn new() -> Self {
        Self { field: "hello" }
    }
}

let mut engine = Engine::new();

engine.register_type::<TestStruct>()
      .register_get_set("xyz", TestStruct::get_field, TestStruct::set_field)
      .register_fn("new_ts", TestStruct::new);

let result = engine.eval::<String>(r#"
                let a = new_ts();
                a.xyz = "42";
                a.xyz
             "#)?;

println!("Answer: {}", result);                     // prints 42
```

**IMPORTANT: Rhai does NOT support normal references (i.e. `&T`) as parameters.**
