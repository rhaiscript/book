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

```admonish warning.side.wide "Custom types are slower"

Custom types run _slower_ than [built-in types][standard types] due to an additional
level of indirection, but for all other purposes there is no difference.
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
