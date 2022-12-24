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

```rust
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


What About Indexers?
--------------------

Many users are tempted to register [indexers] for custom collections.  This essentially makes the
original Rust type something similar to `Vec<MyType>`.

Rhai's standard [`Array`] type is `Vec<Dynamic>` which already holds an ordered, iterable and
indexable collection of dynamic items.  Since Rhai has built-in support, manipulating [arrays] is fast.

In _most_ circumstances, it is better to use [`Array`] instead of a [custom type].

~~~admonish tip.small "Tip: Convert to `Array` using `.into()`"

[`Dynamic`] implements `FromIterator` for all iterable types and an [`Array`] is created in the process.

So, converting a typed array (i.e. `Vec<MyType>`) into an [array] in Rhai is as simple as calling `.into()`.

```rust
// Say you have a custom typed array...
let my_custom_array: Vec<MyType> = do_lots_of_calc(42);

// Convert it into a 'Dynamic' that holds an array
let value: Dynamic = my_custom_array.into();

// Use is anywhere in Rhai...
scope.push("my_custom_array", value);

engine
    // Raw function that returns a custom type
    .register_fn("do_lots_of_calc_raw", do_lots_of_calc)
    // Wrap function that return a custom typed array
    .register_fn("do_lots_of_calc", |seed: i64| -> Dynamic {
        let result = do_lots_of_calc(seed);     // Vec<MyType>
        result.into()                           // Array in Dynamic
    });
```
~~~


TL;DR
-----

~~~admonish question "Why shouldn't we register `Vec<MyType>`?"

### Reason #1: Performance

A main reason why anybody would want to do this is to avoid the overheads of storing [`Dynamic`] items.

This is why [BLOB's] is a built-in data type in Rhai, even though it is actually defined as `Vec<u8>`.
The overhead of using [`Dynamic`] (16 bytes) versus `u8` (1 byte) is worth the trouble, although the
performance gains may not be as pronounced as expected: benchmarks show a 15% speed improvement inside
a tight loop compared with using an [array].

`Vec<MyType>`, however, will be treated as an opaque [custom type] in Rhai, so performance is not optimized.
What you gain from avoiding [`Dynamic`], you pay back in terms of slower access to the `Vec` as well as `MyType`
(which is treated as yet another opaque [custom type]).

### Reason #2: API

Another reason why it shouldn't be done is due to the large number of functions and methods that must be registered
for each type of this sort.  One only has to look at the vast API surface of [arrays]({{rootUrl}}/language/arrays.md#built-in-functions)
to see the common methods that a user would expect to be available.

Since `Vec<Type>` looks, feels and quacks just like a normal [array], and the usage syntax is almost equivalent (except
for the fact that the data type is restricted), users would be frustrated if they find that certain functions available for
[arrays] are not provided.

This is similar to JavaScript's [_Typed Arrays_](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Typed_arrays).
They are quite awkward to work with, and basically each has a full API definition that must be pre-registered.
~~~

