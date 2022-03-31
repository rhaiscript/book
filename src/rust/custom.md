Register any Rust Type and its Methods
=====================================

{{#include ../links.md}}


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


Register a Custom Type and its Methods
-------------------------------------

```admonish tip.side.wide "Tip: Working with enums"

It is also possible to use Rust enums with Rhai.

See the pattern [Working with Enums]({{rootUrl}}/patterns/enums.md) for more details.
```

Any custom type must implement the `Clone` trait as this allows the [`Engine`] to pass by value.

If the [`sync`] feature is used, it must also be `Send + Sync`.

Notice that the custom type needs to be _registered_ using `Engine::register_type`
or `Engine::register_type_with_name`.

To use native methods on custom types in Rhai scripts, it is common to register an API for the type
via the `Engine::register_XXX` API.

```rust
use rhai::{Engine, EvalAltResult};

#[derive(Debug, Clone)]
struct TestStruct {
    field: i64
}

impl TestStruct {
    fn new() -> Self {
        Self { field: 1 }
    }

    fn update(&mut self, x: i64) {      // methods take &mut as first parameter
        self.field += x;
    }
}

let mut engine = Engine::new();

// Most Engine API's can be chained up.
engine.register_type::<TestStruct>()    // register custom type
      .register_fn("new_ts", TestStruct::new)
      .register_fn("update", TestStruct::update);

// Cast result back to custom type.
let result = engine.eval::<TestStruct>(
"
    let x = new_ts();                   // calls 'TestStruct::new'
    x.update(41);                       // calls 'TestStruct::update'
    x                                   // 'x' holds a 'TestStruct'
")?;

println!("result: {}", result.field);   // prints 42
```


First Parameter Must be `&mut`
-----------------------------

_Methods_ of custom types take a `&mut` first parameter to that type, so that invoking methods can
always update it.

All other parameters in Rhai are passed by value (i.e. clones).

```admonish danger.small "No support for references"

Rhai does NOT support normal references (i.e. `&T`) as parameters.
All references must be mutable (i.e. `&mut T`).
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
