Parse an Object Map from JSON
=============================

{{#include ../links.md}}

Do It Without `serde`
---------------------

```admonish info.side.wide "Object map vs. JSON"

A valid JSON object hash does not start with a hash character `#` while a Rhai [object map] does.
That's the only difference!
```

The syntax for an [object map] is extremely similar to the JSON representation of a object hash,
with the exception of `null` values which can technically be mapped to [`()`].

Use the `Engine::parse_json` method to parse a piece of JSON into an [object map].

```rust
// JSON string - notice that JSON property names are always quoted
//               notice also that comments are acceptable within the JSON string
let json = r#"{
                "a": 1,                 // <- this is an integer number
                "b": true,
                "c": 123.0,             // <- this is a floating-point number
                "$d e f!": "hello",     // <- any text can be a property name
                "^^^!!!": [1,42,"999"], // <- value can be array or another hash
                "z": null               // <- JSON 'null' value
              }"#;

// Parse the JSON expression as an object map
// Set the second boolean parameter to true in order to map 'null' to '()'
let map = engine.parse_json(json, true)?;

map.len() == 6;       // 'map' contains all properties in the JSON string

// Put the object map into a 'Scope'
let mut scope = Scope::new();
scope.push("map", map);

let result = engine.eval_with_scope::<i64>(&mut scope, r#"map["^^^!!!"].len()"#)?;

result == 3;          // the object map is successfully used in the script
```

```admonish warning.small "Warning: Must be object hash"

The JSON text must represent a single object hash &ndash; i.e. must be wrapped within braces
`{`...`}`.

It cannot be a primitive type (e.g. number, string etc.).
Otherwise it cannot be converted into an [object map] and a type error is returned.
```

```admonish note.small "Representation of numbers"

JSON numbers are all floating-point while Rhai supports integers (`INT`) and floating-point (`FLOAT`)
(except under [`no_float`]).

Most common generators of JSON data distinguish between integer and floating-point values by always
serializing a floating-point number with a decimal point (i.e. `123.0` instead of `123` which is
assumed to be an integer).

This style can be used successfully with Rhai [object maps].
```

### Sub-objects

Sub-objects are handled transparently by `Engine::parse_json`.

It is _not_ necessary to replace `{` with `#{` in order to fake a Rhai [object map] literal.

```rust
// JSON with sub-object 'b'.
let json = r#"{"a":1, "b":{"x":true, "y":false}}"#;

// 'parse_json' handles this just fine.
let map = engine.parse_json(json, false)?;

// 'map' contains two properties: 'a' and 'b'
map.len() == 2;
```

```admonish question.small "TL;DR &ndash; How is it done?"

Internally, `Engine::parse_json` _cheats_ by treating the JSON text as a Rhai script.

That is why it even supports [comments] and arithmetic expressions in the JSON text.

A [token remap filter] is used to convert `{` into `#{` and `null` to [`()`].
```


Use `serde`
-----------

```admonish info.side "See also"

See _[Serialization/ Deserialization of `Dynamic` with `serde`][`serde`]_ for more details.
```

Remember, `Engine::parse_json` is nothing more than a _cheap_ alternative to true JSON parsing.

If strict correctness is needed, or for more configuration possibilities, turn on the
[`serde`][features] feature to pull in [`serde`](https://crates.io/crates/serde) which enables
serialization and deserialization to/from multiple formats, including JSON.

Beware, though... the [`serde`](https://crates.io/crates/serde) crate is quite heavy.
