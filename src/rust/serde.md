Serialization and Deserialization of `Dynamic` with `serde`
=========================================================

{{#include ../links.md}}

[`serde`]: https://crates.io/crates/serde
[`serde::Serialize`]: https://docs.serde.rs/serde/trait.Serialize.html
[`serde::Deserialize`]: https://docs.serde.rs/serde/trait.Deserialize.html


Rhai's [`Dynamic`] type supports serialization and deserialization by
[`serde`](https://crates.io/crates/serde) via the [`serde`][features] feature.

[`Dynamic`] works _both_ as a _serialization format_ as well as a data type that is serializable.


Serialize/Deserialize a `Dynamic`
--------------------------------

With the [`serde`][features] feature turned on, [`Dynamic`] implements [`serde::Serialize`] and
[`serde::Deserialize`], so it can easily be serialized and deserialized with [`serde`].

```rust,no_run
let value: Dynamic = ...;

// Serialize 'Dynamic' to JSON
let json = serde_json::to_string(&value);

// Deserialize 'Dynamic' from JSON
let result: Dynamic = serde_json::from_str(&json);
```

[Custom types] are serialized as text strings of the value's type name.


`Dynamic` as Serialization Format
--------------------------------

A [`Dynamic`] can be seamlessly converted to and from any type that implements [`serde::Serialize`]
and/or [`serde::Deserialize`], acting as a serialization format.

### Serialize Any Type to `Dynamic`

The function `rhai::serde::to_dynamic` automatically converts any Rust type that implements
[`serde::Serialize`] into a [`Dynamic`].

For primary types, this is usually not necessary because using [`Dynamic::from`][`Dynamic`] is much
easier and is essentially the same thing.  The only difference is treatment for integer values.
`Dynamic::from` keeps different integer types intact, while `rhai::serde::to_dynamic` converts them
all into [`INT`][standard types] (i.e. the system integer type which is `i64` or `i32` depending on
the [`only_i32`] feature).

Rust `struct`'s (or any type that is marked as a `serde` map) are converted into [object maps] while
Rust `Vec`'s (or any type that is marked as a `serde` sequence) are converted into [arrays].

While it is also simple to serialize a Rust type to `JSON` via `serde`,
then use [`Engine::parse_json`]({{rootUrl}}/language/json.md) to convert it into an [object map],
`rhai::serde::to_dynamic` serializes it to [`Dynamic`] directly via `serde` without going through the `JSON` step.

```rust,no_run
use rhai::{Dynamic, Map};
use rhai::serde::to_dynamic;

#[derive(Debug, serde::Serialize)]
struct Point {
    x: f64,
    y: f64
}

#[derive(Debug, serde::Serialize)]
struct MyStruct {
    a: i64,
    b: Vec<String>,
    c: bool,
    d: Point
}

let x = MyStruct {
    a: 42,
    b: vec![ "hello".into(), "world".into() ],
    c: true,
    d: Point { x: 123.456, y: 999.0 }
};

// Convert the 'MyStruct' into a 'Dynamic'
let map: Dynamic = to_dynamic(x);

map.is::<Map>() == true;
```

### Deserialize a `Dynamic` into Any Type

The function `rhai::serde::from_dynamic` automatically converts a [`Dynamic`] value into any Rust type
that implements [`serde::Deserialize`].

In particular, [object maps] are converted into Rust `struct`'s (or any type that is marked as
a `serde` map) while [arrays] are converted into Rust `Vec`'s (or any type that is marked
as a `serde` sequence).

```rust,no_run
use rhai::{Engine, Dynamic};
use rhai::serde::from_dynamic;

#[derive(Debug, serde::Deserialize)]
struct Point {
    x: f64,
    y: f64
}

#[derive(Debug, serde::Deserialize)]
struct MyStruct {
    a: i64,
    b: Vec<String>,
    c: bool,
    d: Point
}

let engine = Engine::new();

let result: Dynamic = engine.eval(
r#"
    #{
        a: 42,
        b: [ "hello", "world" ],
        c: true,
        d: #{ x: 123.456, y: 999.0 }
    }
"#)?;

// Convert the 'Dynamic' object map into 'MyStruct'
let x: MyStruct = from_dynamic(&result)?;
```

### Cannot Deserialize Shared Values

A [`Dynamic`] containing a _shared_ value cannot be deserialized &ndash; i.e. it will give a type error.

Use `Dynamic::flatten` to obtain a cloned copy before deserialization
(if the value is not shared, it is simply returned and not cloned).

Shared values are turned off via the [`no_closure`] feature.


Lighter Alternative
-------------------

The [`serde`](https://crates.io/crates/serde) crate is quite heavy.

If only _simple_ JSON parsing (i.e. only deserialization) of a hash object into a Rhai [object map] is required,
the [`Engine::parse_json`]({{rootUrl}}/language/json.md}}) method is available as a _cheap_ alternative,
but it does not provide the same level of correctness, nor are there any configurable options.


Working with BLOB's
-------------------

[BLOB's], or byte-arrays, are normally serialized and deserialized as simple [arrays].

For higher efficiency, it is necessary to specify [BLOB] fields via the
`serde_bytes` attribute from the [`serde_bytes`](https://crates.io/crates/serde_bytes) crate.

```rust,no_run
use serde::{Deserialize, Serialize};

// Use 'serde_bytes' to serialize the data as Dynamic BLOB's
#[derive(Deserialize, Serialize)]
struct TypeWithBlobs<'a> {
    #[serde(with = "serde_bytes")]
    bytes: &'a [u8],

    #[serde(with = "serde_bytes")]
    byte_buf: Vec<u8>,
}

let blobs = from_dynamic::<TypeWithBlobs>(&blob)?;

// Use 'serde_bytes::Bytes' to get a slice to a stream of bytes
let bytes_ref: &[u8] = from_dynamic::<serde_bytes::Bytes>(&blob)?.as_ref();

// Use 'serde_bytes::ByteBuf' to get a 'Vec<u8>'
let bytes: Vec<u8> = from_dynamic::<serde_bytes::ByteBuf>(&blob)?.into_vec();
```
