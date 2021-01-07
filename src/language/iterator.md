Iterators for Custom Types
==========================

{{#include ../links.md}}

If a [custom type] is iterable, the [`for`](for.md) loop can be used to iterate through
its items in sequence.

In order to use a [`for`](for.md) statement, a _type iterator_ must be registered for
the [custom type] in question.

`Engine::register_iterator<T>` allows registration of a _type iterator_ for any type
that implements `IntoIterator`:

```rust
// Custom type
#[derive(Debug, Clone)]
struct TestStruct { ... }

// Implement 'IntoIterator' trait
impl IntoIterator<Item = ...> for TestStruct {
    type Item = ...;
    type IntoIter = SomeIterType<Self::Item>;

    fn into_iter(self) -> Self::IntoIter {
        ...
    }
}

engine
    .register_type_with_name::<TestStruct>("TestStruct")
    .register_fn("new_ts", || TestStruct { ... })
    .register_iterator::<TestStruct>();           // register type iterator
```

With a type iterator registered, the [custom type] can be iterated through:

```rust
let ts = new_ts();

// Use 'for' statement to loop through items in 'ts'
for item in ts {
    ...
}
```
