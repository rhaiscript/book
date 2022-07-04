Working with Any Rust Type
===========================

{{#include ../links.md}}

```admonish tip.side.wide "Tip: Shared types"

The only requirement of a type to work with Rhai is `Clone`.

Therefore, it is extremely easy to use Rhai with data types such as
`Rc<...>`, `Arc<...>`, `Rc<RefCell<...>>`, `Arc<Mutex<...>>` etc.
```

~~~admonish note.side.wide "Under `sync`"

If the [`sync`] feature is used, a custom type must also be `Send + Sync`.
~~~

Rhai works seamlessly with any Rust type, as long as it implements `Clone` as this allows the
[`Engine`] to pass by value.

A type that is not one of the [standard types] is termed a "custom type".

Custom types can have the following:

* a custom (friendly) display name

* [methods]

* [property getters and setters](getters/setters)

* [indexers]


Free Typing
-----------

```admonish question.side.wide "Why \\"Custom\\"?"

Rhai internally supports a number of standard data types (see [this list][standard types]).

Any type outside of the list is considered _custom_.
```

Rhai works seamlessly with _any_ Rust type.

A custom type is stored in Rhai as a Rust _trait object_ (specifically, a `dyn rhai::Variant`),
with no restrictions other than being `Clone` (plus `Send + Sync` under the [`sync`] feature).

The type literally does not have any prerequisite other than being `Clone`.

It does not need to implement any other trait or use any custom `#[derive]`.

This allows Rhai to be integrated into an existing Rust code base with as little plumbing as
possible, usually silently and seamlessly.

External types that are not defined within the same crate (and thus cannot implement special Rhai
traits or use special `#[derive]`) can also be used easily with Rhai.

Support for custom types can be turned off via the [`no_object`] feature.

```admonish warning.small "Custom types are slower"

Custom types run _slower_ than [built-in types][standard types] due to an additional
level of indirection, but for all other purposes there is no difference.
```


Register a Custom Type
----------------------

The custom type needs to be _registered_ using `Engine::register_type` or
`Engine::register_type_with_name`.

```rust
use rhai::{Engine, EvalAltResult};

#[derive(Debug, Clone)]
struct TestStruct {
    field: i64
}

let mut engine = Engine::new();

// Register custom type with friendly  name
engine.register_type_with_name::<TestStruct>("TestStruct");
```

```admonish tip.small "Tip: Working with enums"

It is also possible to use Rust enums with Rhai.

See the pattern [Working with Enums]({{rootUrl}}/patterns/enums.md) for more details.
```


`type_of()` a Custom Type
-------------------------

[`type_of()`] works fine with custom types and returns the name of the type.

If `Engine::register_type_with_name` is used to register the custom type with a special
"pretty-print" name, [`type_of()`] will return that name instead.

```rust
engine.register_type::<TestStruct1>()
      .register_fn("new_ts1", TestStruct1::new)
      .register_type_with_name::<TestStruct2>("TestStruct")
      .register_fn("new_ts2", TestStruct2::new);

let ts1_type = engine.eval::<String>("let x = new_ts1(); x.type_of()")?;
let ts2_type = engine.eval::<String>("let x = new_ts2(); x.type_of()")?;

println!("{}", ts1_type);               // prints 'path::to::TestStruct'
println!("{}", ts1_type);               // prints 'TestStruct'
```


`==` Operator
-------------

Many standard functions (e.g. filtering, searching and sorting) expect a custom type to be
_comparable_, meaning that the `==` operator must be registered for the custom type.

For example, in order to use the [`in`] operator with a custom type for an [array],
the `==` operator is used to check whether two values are the same.

```rust
// Assume 'TestStruct' implements `PartialEq`
engine.register_fn("==",
    |item1: &mut TestStruct, item2: TestStruct| item1 == &item2
);

// Then this works in Rhai:
let item = new_ts();        // construct a new 'TestStruct'
item in array;              // 'in' operator uses '=='
```
