Make a Custom Type Iterable
===========================

{{#include ../links.md}}

```admonish info.side "Built-in type iterators"

Type iterators are already defined for built-in [standard types] such as [strings], [ranges],
[bit-fields], [arrays] and [object maps].

That's why they can be used with the [`for`] loop.
```

If a [custom type] is iterable, the [`for`] loop can be used to iterate through
its items in sequence, as long as it has a _type iterator_ registered.

`Engine::register_iterator<T>` allows registration of a type iterator for any type
that implements `IntoIterator`.

With a type iterator registered, the [custom type] can be iterated through.

```rust
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

// 'TestStruct' is now iterable
engine.run(
"
    for value in new_ts() {
        ...
    }
")?;
```

```admonish tip.small "Tip: Fallible type iterators"

`Engine::register_iterator_result` allows registration of a _fallible_ type iterator &ndash;
i.e. an iterator that returns `Result<T, Box<EvalAltResult>>`.

On in very rare situations will this be necessary though.
```
