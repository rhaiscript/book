Register any Rust Type and its Methods
=====================================

{{#include ../links.md}}


Free Typing
-----------

Rhai works seamlessly with _any_ Rust type.  The type can be _anything_; it does not
have any prerequisites other than being `Clone`.  It does not need to implement
any other trait or use any custom `#[derive]`.

This allows Rhai to be integrated into an existing Rust code base with as little plumbing
as possible, usually silently and seamlessly.  External types that are not defined
within the same crate (and thus cannot implement special Rhai traits or
use special `#[derive]`) can also be used easily with Rhai.

The reason why it is termed a _custom_ type throughout this documentation is that
Rhai natively supports a number of data types with fast, internal treatment (see
the list of [standard types]).  Any type outside of this list is considered _custom_.

Any type not supported natively by Rhai is stored as a Rust _trait object_, with no
restrictions other than being `Clone` (plus `Send + Sync` under the [`sync`] feature).
It runs slightly slower than natively-supported types as it does not have built-in,
optimized implementations for commonly-used functions, but for all other purposes has
no difference.

Support for custom types can be turned off via the [`no_object`] feature.


Register a Custom Type and its Methods
-------------------------------------

Any custom type must implement the `Clone` trait as this allows the [`Engine`] to pass by value.

If the [`sync`] feature is used, it must also be `Send + Sync`.

Notice that the custom type needs to be _registered_ using `Engine::register_type`
or `Engine::register_type_with_name`.

To use native methods on custom types in Rhai scripts, it is common to register an API
for the type via the `Engine::register_XXX` methods.

```rust , no_run
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
let result = engine.eval::<TestStruct>(r"
                let x = new_ts();       // calls 'TestStruct::new'
                x.update(41);           // calls 'TestStruct::update'
                x                       // 'x' holds a 'TestStruct'
             ")?;

println!("result: {}", result.field);   // prints 42
```

Rhai follows the convention that methods of custom types take a `&mut` first parameter
to that type, so that invoking methods can always update it.

All other parameters in Rhai are passed by value (i.e. clones).

**IMPORTANT: Rhai does NOT support normal references (i.e. `&T`) as parameters.**


`type_of()` a Custom Type
-------------------------

[`type_of()`] works fine with custom types and returns the name of the type.

If `Engine::register_type_with_name` is used to register the custom type
with a special "pretty-print" name, [`type_of()`] will return that name instead.

```rust , no_run
engine.register_type::<TestStruct1>()
      .register_fn("new_ts1", TestStruct1::new)
      .register_type_with_name::<TestStruct2>("TestStruct")
      .register_fn("new_ts2", TestStruct2::new);

let ts1_type = engine.eval::<String>(r#"let x = new_ts1(); x.type_of()"#)?;
let ts2_type = engine.eval::<String>(r#"let x = new_ts2(); x.type_of()"#)?;

println!("{}", ts1_type);               // prints 'path::to::TestStruct'
println!("{}", ts1_type);               // prints 'TestStruct'
```


Collection Types
----------------

Collection types usually contain a `push`, `insert`, `add`, `append` or `+=` method that adds a particular
item to the collection.

If the collection takes a [`Dynamic`] value (e.g. like an [array]), the type of such an add function
can take a [`Dynamic`] parameter.

```rust , no_run
engine.register_fn("push",
    |col: &mut MyCollectionType, item: Dynamic| col.push(col)
);
```


Use the Custom Type With Arrays
------------------------------

In order to use the [`in`] operator with a custom type for an [array], the `==` operator must be
registered for the custom type:

```rust , no_run
// Assume 'TestStruct' implements `PartialEq`
engine.register_fn("==",
    |item1: &mut TestStruct, item2: TestStruct| item1 == &item2
);

// Then this works in Rhai:
let item = new_ts();        // construct a new 'TestStruct'
item in array;              // 'in' operator uses '=='
```


Working With Enums
------------------

It is quite easy to use Rust enums with Rhai.
See the section on [Working with Enums]({{rootUrl}}/patterns/enums.md) for more details.
