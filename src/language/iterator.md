Make a Custom Type Iterable
===========================

{{#include ../links.md}}

If a [custom type] is iterable, the [`for`] loop can be used to iterate through
its items in sequence, as long as it has a _type iterator_ registered.

Type iterators are already defined for built-in [standard types] such as [strings], [ranges],
[bit-fields], [arrays] and [object maps]. That's why they can be used with the [`for`] loop.

`Engine::register_iterator<T>` allows registration of a type iterator for any type
that implements `IntoIterator`.

With a type iterator registered, the [custom type] can be iterated through.

```rust,no_run
// Custom type
#[derive(Debug, Clone)]
struct TestStruct { fields: Vec<i64> }

// Implement 'IntoIterator' trait
impl IntoIterator<Item = i64> for TestStruct {
    type Item = i64;
    type IntoIter = std::vec::IntoIter<Self::Item>;

    fn into_iter(self) -> Self::IntoIter {
        self.fields.into_iter()
    }
}

let mut engine = Engine::new();

// Register API and type iterator for 'TestStruct'
engine.register_type_with_name::<TestStruct>("TestStruct")
      .register_fn("new_ts", || TestStruct { fields: vec![1, 2, 3, 42] })
      .register_iterator::<TestStruct>();

engine.run(
"
    // 'TestStruct' is iterable
    let ts = new_ts();

    // Use 'for' statement to loop through items in 'ts'
    for value in ts {
        ...
    }
")?;
```
