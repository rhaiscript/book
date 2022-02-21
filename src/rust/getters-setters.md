Custom Type Property Getters and Setters
=======================================

{{#include ../links.md}}

```admonish warning.side-wide-narrow "Cannot override object maps"

Property getters and setters are intended for [custom types].

Any getter or setter function registered for [object maps] is simply _ignored_.

Get/set syntax on [object maps] is interpreted as access to properties.
```

A [custom type] can also expose properties by registering `get` and/or `set` functions.

Properties can be accessed in a Rust-like syntax:

> _object_ `.` _property_
>
> _object_ `.` _property_ `=` _value_ `;`

Property getter and setter functions are called behind the scene.
They each take a `&mut` reference to the first parameter.

Getters and setters are disabled under the [`no_object`] feature.

<section></section>

| `Engine` API          | Function signature(s)<br/>(`T: Clone` = custom type,<br/>`V: Clone` = data type) |        Can mutate `T`?         |
| --------------------- | -------------------------------------------------------------------------------- | :----------------------------: |
| `register_get`        | `Fn(&mut T) -> V`                                                                |      yes, but not advised      |
| `register_set`        | `Fn(&mut T, V)`                                                                  |              yes               |
| `register_get_set`    | getter: `Fn(&mut T) -> V`</br>setter: `Fn(&mut T, V)`                            | yes, but not advised in getter |
| `register_get_result` | `Fn(&mut T) -> Result<V, Box<EvalAltResult>>`                                    |      yes, but not advised      |
| `register_set_result` | `Fn(&mut T, V) -> Result<(), Box<EvalAltResult>>`                                |              yes               |

```admonish danger "No support for references"

Rhai does NOT support normal references (i.e. `&T`) as parameters.
All references must be mutable (i.e. `&mut T`).
```

```admonish warning "Getters must be pure"

By convention, property getters are assumed to be _pure_, meaning that they are not supposed to
mutate the [custom type], although there is nothing that prevents this mutation in Rust.

Even though a property getter function also takes `&mut` as the first parameter, Rhai assumes that
no data is changed when the function is called.
```


Examples
--------

```rust,no_run
#[derive(Debug, Clone)]
struct TestStruct {
    field: String
}

impl TestStruct {
    // Remember &mut must be used even for getters.
    fn get_field(&mut self) -> String {
        // Property getters are assumed to be PURE, meaning they are
        // not supposed to mutate any data.
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

let result = engine.eval::<String>(
r#"
    let a = new_ts();
    a.xyz = "42";
    a.xyz
"#)?;

println!("Answer: {}", result);                 // prints 42
```


Fallback to Indexer
-------------------

If the getter/setter of a particular property is not defined, but an [indexer] is defined on the
[custom type] with [string] index, then the corresponding [indexer] will be called with the name of
the property as the index value.

In other words, [indexers] act as a _fallback_ to property getters/setters.

```rust,no_run
a.foo           // if property getter for 'foo' doesn't exist...

a["foo"]        // an indexer (if any) is tried
```

This feature makes it very easy for [custom types] to act as _property bags_
(similar to an [object map]) which can add/remove properties at will.


Chaining Updates
----------------

It is possible to _chain_ property accesses and/or indexing (via [indexers]) together to modify a
particular property value at the end of the chain.

Rhai detects such modifications and updates the changed values all the way back up the chain.

In the end, the syntax works as expected by intuition, automatically and without special attention.

```rust,no_run
// Assume a deeply-nested object...
let root = get_new_container_object();

root.prop1.sub["hello"].list[0].value = 42;

// The above is equivalent to:

// First getting all the intermediate values...
let prop1_value = root.prop1;                   // via property getter
let sub_value = prop1_value.sub;                // via property getter
let sub_value_item = sub_value["hello"];        // via index getter
let list_value = sub_value_item.list;           // via property getter
let list_item = list_value[0];                  // via index getter

list_item.value = 42;       // modify property value deep down the chain

// Propagate the changes back up the chain...
list_value[0] = list_item;                      // via index setter
sub_value_item.list = list_value;               // via property setter
sub_value["hello"] = sub_value_item;            // via index setter
prop1_value.sub = sub_value;                    // via property setter
root.prop1 = prop1_value;                       // via property setter

// The below prints 42...
print(root.prop1.sub["hello"].list[0].value);
```
