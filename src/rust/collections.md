Custom Collection Types
=======================

{{#include ../links.md}}

~~~admonish tip.side "Tip"

Collections can also hold [`Dynamic`] values (e.g. like an [array]).
~~~

A _collection_ type holds a... well... _collection_ of items.  It can be homogeneous (all items are
the same type) or heterogeneous (items are of different types, use [`Dynamic`] to hold).

Because their only purpose for existence is to hold a number of items, collection types commonly
register the following methods.

<section></section>

| Method                    | Description                                                      |
| ------------------------- | ---------------------------------------------------------------- |
| `len` method and property | gets the total number of items in the collection                 |
| `clear`                   | clears the collection                                            |
| `contains`                | checks if a particular item exists in the collection             |
| `add`, `+=` operator      | adds a particular item to the collection                         |
| `remove`, `-=` operator   | removes a particular item from the collection                    |
| `merge` or `+` operator   | merges two collections, yielding a new collection with all items |

```admonish tip.small "Tip: Define type iterator"

Collections are typically iterable.

It is customary to use `Engine::register_iterator` to allow iterating the collection if
it implements `IntoIterator`.

Alternative, register a specific [type iterator] for the [custom type].
```

```admonish tip.small "Tip: Use a plugin module"

A [plugin module] makes defining an entire API for a [custom type] a snap.
```


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
