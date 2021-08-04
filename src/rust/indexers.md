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

By convention, index [getters][getters/setters] are not supposed to mutate the [custom type],
although there is nothing that prevents this mutation.

**IMPORTANT: Rhai does NOT support normal references (i.e. `&T`) as parameters.**


Cannot Override Arrays, Object Maps, Strings and Integers
--------------------------------------------------------

For efficiency reasons, indexers **cannot** be used to overload (i.e. override)
built-in indexing operations for [arrays], [object maps], [strings] and integers
(acting as [bit-field] operation).

These types have built-in indexer implementations that are fast and efficient:

| Type                                      |                Index type                 | Return type | Description                                                 |
| ----------------------------------------- | :---------------------------------------: | :---------: | ----------------------------------------------------------- |
| [`Array`]                                 |                   `INT`                   | [`Dynamic`] | access a particular element inside the [array]              |
| [`Map`]                                   | [`ImmutableString`],<br/>`String`, `&str` | [`Dynamic`] | access a particular property inside the [object map]        |
| [`ImmutableString`],<br/>`String`, `&str` |                   `INT`                   | [character] | access a particular [character] inside the [string]         |
| `INT`                                     |                   `INT`                   |   boolean   | access a particular bit inside the integer as a [bit-field] |

Attempting to register indexers for an [array], [object map], [string] or `INT` panics
when using the `Engine::register_indexer_XXX` API.  They can, however, be defined in a
[plugin module], only to be ignored.

In general, it is a bad idea to overload indexers for any of the [standard types] supported
internally by Rhai, since built-in indexers may be added in future versions.


Convention for Negative Index
-----------------------------

If the indexer takes a signed integer as an index (e.g. the standard `INT` type), care should be
taken to handle _negative_ values passed as the index.

It is a standard API _convention_ for Rhai to assume that an index position counts _backwards_ from
the _end_ if it is negative.

`-1` as an index usually refers to the _last_ item, `-2` the second to last item, and so on.

Therefore, negative index values go from `-1` (last item) to `-length` (first item).

A typical implementation for negative index values is:

```rust , no_run
// The following assumes:
//   'index' is 'INT', 'items_len: usize' is the number of elements
let actual_index = if index < 0 {
    index.checked_abs().map_or(0, |n| items_len - (n as usize).min(items_len))
} else {
    index as usize
};
```

The _end_ of a data type can be interpreted creatively.  For example, in an integer used as a
[bit-field], the _start_ is the _least-significant-bit_ (LSB) while the `end` is the
_most-significant-bit_ (MSB).


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

let result = engine.eval::<i64>(
r#"
    let a = new_ts();
    a["xyz"] = 42;                  // these indexers use strings
    a["xyz"]                        // as the index type
"#)?;

println!("Answer: {}", result);     // prints 42
```


Indexer as Property Access Fallback
----------------------------------

An indexer taking a [string] index is a special case.  It acts as a _fallback_ to property
[getters/setters].

During a property access, if the appropriate property [getter/setter][getters/setters] is not
defined, an indexer is called and passed the string name of the property.

This is also extremely useful as a short-hand for indexers, when the [string] keys conform to
property name syntax.

```rust , no_run
// You can write this...
let x = foo["hello_world"];

// but it is easier with this...
let x = foo.hello_world;
```

The reverse, however, is not true &ndash; when an indexer fails or doesn't exist, the corresponding
property [getter/setter][getters/setters], if any, is not called.

```rust , no_run
type MyType = HashMap<String, i64>;

let mut engine = Engine::new();

// Define custom type, property getter and string indexers
engine.register_type::<MyType>()
      .register_fn("new_ts", || {
          let mut object = MyType::new();
          object.insert("foo", 1);
          object.insert("bar", 42);
          object.insert("baz", 123);
      })
      // Property 'hello'
      .register_get("hello", |object: &mut MyType| object.len() as i64)
      // Index getter/setter
      .register_indexer_get(|object: &mut MyType, index: &str| *object[index])
      .register_indexer_set(|object: &mut MyType, index: &str, value: i64| object[index] = value);

// Calls a["foo"] because getter for 'foo' does not exist
engine.run("let a = new_ts(); print(a.foo);");

// Calls a["bar"] because getter for 'bar' does not exist
engine.run("let a = new_ts(); print(a.bar);");

// Calls a["baz"] = 999 because getter for 'baz' does not exist
engine.run("let a = new_ts(); a.baz = 999;");

// Error: Property getter is not a fallback for indexer
engine.run(r#"let a = new_ts(); print(a["hello"]);"#);
```
