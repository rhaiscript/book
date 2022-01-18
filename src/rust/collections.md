Custom Collection Types
=======================

{{#include ../links.md}}

A _collection_ type holds a... well... _collection_ of items.  It can be homogeneous (all items are
the same type) or heterogeneous (items are of different types, use [`Dynamic`] to hold).

Because their only purpose for existence is to hold a number of items, collection types commonly
register the following methods.

| Method                    | Description                                                      |
| ------------------------- | ---------------------------------------------------------------- |
| `len` method and property | gets the total number of items in the collection                 |
| `clear`                   | clears the collection                                            |
| `contains`                | checks if a particular item exists in the collection             |
| `add`, `+=` operator      | adds a particular item to the collection                         |
| `remove`, `-=` operator   | removes a particular item from the collection                    |
| `merge` or `+` operator   | merges two collections, yielding a new collection with all items |

If the collection takes a [`Dynamic`] value (e.g. like an [array]), these functions can take
[`Dynamic`] parameters.

Furthermore, it is customary to use `Engine::register_iterator` to allow iterating the collection if
it implements `IntoIterator`.


Example
-------

```rust,no_run
type MyBag = HashSet<MyItem>;

engine
    .register_type_with_name::<MyBag>("MyBag")
    .register_iterator::<MyBag>()
    .register_fn("new_bag", || MyBag::new())
    .register_fn("len", |col: &mut MyBag| col.len() as i64)
    .register_get("len", |col: &mut MyBag| col.len() as i64)
    .register_fn("clear", |col: &mut MyBag| col.clear())
    .register_fn("contains", |col: &mut MyBag, item: i64| col.contains(&item))
    .register_fn("add", |col: &mut MyBag, item: MyItem| col.insert(item))
    .register_fn("+=", |col: &mut MyBag, item: MyItem| col.insert(item))
    .register_fn("remove", |col: &mut MyBag, item: MyItem| col.remove(&item))
    .register_fn("-=", |col: &mut MyBag, item: MyItem| col.remove(&item))
    .register_fn("+", |mut col1: MyBag, col2: MyBag| {
        col1.extend(col2.into_iter());
        col1
    });
```
