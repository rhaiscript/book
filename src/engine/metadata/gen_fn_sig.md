Get Native Function Signatures
==============================

{{#include ../../links.md}}


`Engine::gen_fn_signatures`
--------------------------

As part of a _reflections_ API, `Engine::gen_fn_signatures` returns a list of function _signatures_
(as `Vec<String>`), each corresponding to a particular native function available to that [`Engine`] instance.

> _name_ `(`_param 1_`:`_type 1_`,` _param 2_`:`_type 2_`,` ... `,` _param n_`:`_type n_`) ->` _return type_

The [`metadata`] feature must be used to turn on this API.

### Sources

Functions from the following sources are included, in order:

1. Native Rust functions registered into the global namespace via the `Engine::register_XXX` API
2. _Public_ (i.e. non-[`private`]) functions (native Rust or Rhai scripted) in global sub-modules
   registered via `Engine::register_static_module`.
3. Native Rust functions in external [packages] registered via `Engine::register_global_module`
4. Native Rust functions in [built-in packages] (optional)


Functions Metadata
------------------

Beware, however, that not all function signatures contain parameters and return value information.

### `Engine::register_XXX`

For instance, functions registered via `Engine::register_XXX` contain no information on the names of
parameter because Rust simply does not make such metadata available natively.

Type names, however, _are_ provided.

A function registered under the name `foo` with three parameters.

> `foo(_: i64, _: char, _: &str) -> String`

An [operator] function. Notice that function names do not need to be valid identifiers.

> `+=(_: &mut i64, _: i64)`

A [property setter][getters/setters].
Notice that function names do not need to be valid identifiers.
In this case, the first parameter should be `&mut T` of the custom type and the return value is `()`:

> `set$prop(_: &mut TestStruct, _: i64)`

### Script-Defined Functions

Script-defined [function] signatures contain parameter names.
Since _all_ parameters, as well as the return value, are [`Dynamic`] the types are simply not shown.

> `foo(x, y, z)`

is probably defined simply as:

```rust
/// This is a doc-comment, included in this function's metadata.
fn foo(x, y, z) {
    ...
}
```

which is really the same as:

> `foo(x: Dynamic, y: Dynamic, z: Dynamic) -> Result<Dynamic, Box<EvalAltResult>>`

### Plugin Functions

Functions defined in [plugin modules] are the best.
They contain all metadata describing the functions, including [doc-comments].

For example, a plugin function `combine`:

> `/// This is a doc-comment, included in this function's metadata.`  
> `combine(list: &mut MyStruct<i64>, num: usize, name: &str) -> bool`

Notice that function names do not need to be valid identifiers.

For example, an [operator] defined as a [fallible function] in a [plugin module] via
`#[rhai_fn(name="+=", return_raw)]` returns `Result<bool, Box<EvalAltResult>>`:

> `+=(list: &mut MyStruct<i64>, value: &str) -> Result<bool, Box<EvalAltResult>>`

For example, a [property getter][getters/setters] defined in a [plugin module]:

> `get$prop(obj: &mut MyStruct<i64>) -> String`
