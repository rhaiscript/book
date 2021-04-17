Custom Type Indexers
===================

{{#include ../links.md}}

A [custom type] can also expose an _indexer_ by registering an indexer function.

A [custom type] with an indexer function defined can use the bracket notation to get/set a property
value at a particular index:

> _object_ `[` _index_ `]`
>
> _object_ `[` _index_ `]` `=` _value_ `;`

Like property [getters/setters], indexers take a `&mut` reference to the first parameter.

They also take an additional parameter of any type that serves as the _index_ within brackets.

Indexers are disabled when the [`no_index`] feature is used.

| `Engine` API                  | Function signature(s)<br/>(`T: Clone` = custom type,<br/>`X: Clone` = index type,<br/>`V: Clone` = data type) |        Can mutate `T`?         |
| ----------------------------- | ------------------------------------------------------------------------------------------------------------- | :----------------------------: |
| `register_indexer_get`        | `Fn(&mut T, X) -> V`                                                                                          |      yes, but not advised      |
| `register_indexer_set`        | `Fn(&mut T, X, V)`                                                                                            |              yes               |
| `register_indexer_get_set`    | getter: `Fn(&mut T, X) -> V`<br/>setter: `Fn(&mut T, X, V)`                                                   | yes, but not advised in getter |
| `register_indexer_get_result` | `Fn(&mut T, X) -> Result<Dynamic, Box<EvalAltResult>>`                                                        |      yes, but not advised      |
| `register_indexer_set_result` | `Fn(&mut T, X, V) -> Result<(), Box<EvalAltResult>>`                                                          |              yes               |

By convention, index getters are not supposed to mutate the [custom type], although there is nothing
that prevents this mutation.

**IMPORTANT: Rhai does NOT support normal references (i.e. `&T`) as parameters.**


Cannot Override Arrays, Object Maps and Strings
----------------------------------------------

For efficiency reasons, indexers **cannot** be used to overload (i.e. override)
built-in indexing operations for [arrays], [object maps] and [strings].

Attempting to register indexers for an [array], [object map] or [string] panics.


Negative Integer Index
----------------------

If the indexer takes a signed integer as an index (e.g. the standard `INT` type), care should be
taken to handle _negative_ values passed as the index.

It is a standard API style for Rhai to assume that an index position counts _backwards_ from the
_end_ if it is negative.

`-1` as an index usually refers to the _last_ item, `-2` the second to last item, and so on.

Therefore, negative index values go from `-1` (last item) to `-length` (first item).

A typical implementation for negative index values is:

```rust , no_run
// The following assumes:
//   'index' is 'INT', 'items_len: usize' is the number of elements
let actual_index: usize = if index < 0 {
    index.checked_abs().map_or(0, |n| items_len - (n as usize).min(items_len))
} else {
    index as usize
};
```


Examples
--------

```rust , no_run
#[derive(Debug, Clone)]
struct TestStruct {
    fields: Vec<i64>
}

impl TestStruct {
    // Remember &mut must be used even for getters
    fn get_field(&mut self, index: String) -> i64 {
        self.fields[index.len()]
    }
    fn set_field(&mut self, index: String, value: i64) {
        self.fields[index.len()] = value
    }

    fn new() -> Self {
        Self { fields: vec![1, 2, 3, 4, 5] }
    }
}

let mut engine = Engine::new();

engine.register_type::<TestStruct>()
      .register_fn("new_ts", TestStruct::new)
      // Short-hand: .register_indexer_get_set(TestStruct::get_field, TestStruct::set_field);
      .register_indexer_get(TestStruct::get_field)
      .register_indexer_set(TestStruct::set_field);

let result = engine.eval::<i64>(r#"
                let a = new_ts();
                a["xyz"] = 42;          // these indexers use strings
                a["xyz"]                // as the index type
             "#)?;

println!("Answer: {}", result);         // prints 42
```
