Builder Pattern / Fluent API
============================

{{#include ../links.md}}


```admonish info "Usage scenario"

* An API uses the [Builder Pattern](https://en.wikipedia.org/wiki/Builder_pattern) or a [fluent API](https://en.wikipedia.org/wiki/Fluent_interface).

* The builder type is not necessarily `Clone`.
```

```admonish abstract "Key concepts"

* Wrap the builder type in shared interior mutability (aka `Rc<RefCell<T>>` or `Arc<RwLock<T>>`).
```

```admonish tip.small "Tip: Fluent API"

This same pattern can be used to implement any [_fluent API_](https://en.wikipedia.org/wiki/Fluent_interface).
```


Implementation With Clonable Builder Type
-----------------------------------------

This assumes that the builder type implements `Clone`.
This is the most common scenario.

```rust
/// Builder for `Foo` instances.
#[derive(Clone)]
pub struct FooBuilder {
    /// The `foo` option.
    foo: i64,
    /// The `bar` option.
    bar: bool,
    /// The `baz` option.
    baz: String,
}

/// `FooBuilder` API which uses moves.
impl FooBuilder {
    /// Creates a new builder for `Foo`.
    pub fn new() -> Self {
        Self { foo: 0, bar: false, baz: String::new() }
    }
    /// Sets the `foo` option.
    pub fn with_foo(mut self, foo: i64) -> Self {
        self.foo = foo;
        self
    }
    /// Sets the `bar` option.
    pub fn with_bar(mut self, bar: bool) -> Self {
        self.bar = bar;
        self
    }
    /// Sets the `baz` option.
    pub fn with_baz(mut self, baz: &str) -> Self {
        self.baz = baz.to_string();
        self
    }
    /// Builds the `Foo` instance.
    pub fn build(self) -> Foo {
        Foo { foo: self.foo, bar: self.bar, baz: self.baz }
    }
}

let mut engine = Engine::new();

engine
    .register_fn("get_foo", FooBuilder::new)
    .register_fn("with_foo", FooBuilder::with_foo)
    .register_fn("with_bar", FooBuilder::with_bar)
    .register_fn("with_baz", FooBuilder::with_baz)
    .register_fn("create", FooBuilder::build);
```


Implementation With Mutable Reference
-------------------------------------

This assumes that the builder type's API uses mutable references.
The builder type does not need to implement `Clone`.

```rust
use rhai::plugin::*;

/// Builder for `Foo` instances.
/// Notice that this type does not need to be `Clone`.
pub struct FooBuilder {
    /// The `foo` option.
    foo: i64,
    /// The `bar` option.
    bar: bool,
    /// The `baz` option.
    baz: String,
}

/// Builder type API uses mutable references.
impl FooBuilder {
    /// Creates a new builder for `Foo`.
    pub fn new() -> Self {
        Self { foo: 0, bar: false, baz: String::new() }
    }
    /// Sets the `foo` option.
    pub fn with_foo(&mut self, foo: i64) -> &mut Self {
        self.foo = foo; self
    }
    /// Sets the `bar` option.
    pub fn with_bar(&mut self, bar: bool) -> &mut Self {
        self.bar = bar; self
    }
    /// Sets the `baz` option.
    pub fn with_baz(&mut self, baz: &str) -> &mut Self {
        self.baz = baz.to_string(); self
    }
    /// Builds the `Foo` instance.
    pub fn build(&self) -> Foo {
        Foo { foo: self.foo, bar: self.bar, baz: self.baz.clone() }
    }
}

/// Builder for `Foo`.
#[export_module]
pub mod foo_builder {
    use super::{Foo, FooBuilder as BuilderImpl};
    use std::cell::RefCell;
    use std::rc::Rc;

    /// The builder for `Foo`.
    // This type is `Clone`.
    pub type FooBuilder = Rc<RefCell<super::BuilderImpl>>;

    /// Creates a new builder for `Foo`.
    pub fn default() -> FooBuilder {
        Rc::new(RefCell::new(BuilderImpl::new()))
    }
    /// Sets the `foo` option.
    #[rhai_fn(global, pure)]
    pub fn with_foo(builder: &mut FooBuilder, foo: i64) -> FooBuilder {
        builder.set_foo(foo);
        builder.clone()
    }
    /// Sets the `bar` option.
    #[rhai_fn(global, pure)]
    pub fn with_bar(builder: &mut FooBuilder, bar: bool) -> FooBuilder {
        builder.set_bar(bar);
        builder.clone()
    }
    /// Sets the `baz` option.
    #[rhai_fn(global, pure)]
    pub fn with_baz(builder: &mut FooBuilder, baz: &str) -> FooBuilder {
        builder.set_baz(baz);
        builder.clone()
    }
    /// Builds the `Foo` instance.
    #[rhai_fn(global, pure)]
    pub fn create(builder: &mut FooBuilder) -> Foo {
        builder.borrow().build()
    }
}
```


Implementation With Moves
-------------------------

What if the builder type's API relies on moves instead of mutable references?
And the builder type does not implement `Clone`?

Not too worry: the following trick has you covered!

```rust
use rhai::plugin::*;

/// Builder for `Foo` instances.
/// Notice that this type does not need to be `Clone`.
pub struct FooBuilder {
    /// The `foo` option.
    foo: i64,
    /// The `bar` option.
    bar: bool,
    /// The `baz` option.
    baz: String,
}

/// `FooBuilder` API which uses moves.
impl FooBuilder {
    /// Creates a new builder for `Foo`.
    pub fn new() -> Self {
        Self { foo: 0, bar: false, baz: String::new() }
    }
    /// Sets the `foo` option.
    pub fn with_foo(mut self, foo: i64) -> Self {
        self.foo = foo;
        self
    }
    /// Sets the `bar` option.
    pub fn with_bar(mut self, bar: bool) -> Self {
        self.bar = bar;
        self
    }
    /// Sets the `baz` option.
    pub fn with_baz(mut self, baz: &str) -> Self {
        self.baz = baz.to_string();
        self
    }
    /// Builds the `Foo` instance.
    pub fn build(self) -> Foo {
        Foo { foo: self.foo, bar: self.bar, baz: self.baz }
    }
}

/// Builder for `Foo`.
#[export_module]
pub mod foo_builder {
    use super::{Foo, FooBuilder as BuilderImpl};
    use std::cell::RefCell;
    use std::mem;
    use std::rc::Rc;

    /// The builder for `Foo`.
    // This type is `Clone`.
    // An `Option` is used for easy extraction of the builder type.
    // If it is `None` then the builder is already consumed.
    pub type FooBuilder = Rc<RefCell<Option<BuilderImpl>>>;

    /// Creates a new builder for `Foo`.
    pub fn default() -> FooBuilder {
        Rc::new(RefCell::new(Some(BuilderImpl::new())))
    }
    /// Sets the `foo` option.
    #[rhai_fn(return_raw, global, pure)]
    pub fn with_foo(builder: &mut FooBuilder, foo: i64) -> Result<FooBuilder, Box<EvalAltResult>> {
        let b = &mut *builder.borrow_mut();

        if let Some(obj) = mem::take(b) {
            *b = Some(obj.with_foo(foo));
            Ok(builder.clone())
        } else {
            Err("Builder is already consumed".into())
        }
    }
    /// Sets the `bar` option.
    #[rhai_fn(return_raw, global, pure)]
    pub fn with_bar(builder: &mut FooBuilder, bar: bool) -> Result<FooBuilder, Box<EvalAltResult>> {
        let b = &mut *builder.borrow_mut();

        if let Some(obj) = mem::take(b) {
            *b = Some(obj.with_bar(bar));
            Ok(builder.clone())
        } else {
            Err("Builder is already consumed".into())
        }
    }
    /// Sets the `baz` option.
    #[rhai_fn(return_raw, global, pure)]
    pub fn with_baz(builder: &mut FooBuilder, baz: &str) -> Result<FooBuilder, Box<EvalAltResult>> {
        let b = &mut *builder.borrow_mut();

        if let Some(obj) = mem::take(b) {
            *b = Some(obj.with_baz(baz));
            Ok(builder.clone())
        } else {
            Err("Builder is already consumed".into())
        }
    }
    /// Builds the `Foo` instance.
    #[rhai_fn(return_raw, global, pure)]
    pub fn create(builder: &mut FooBuilder) -> Result<Foo, Box<EvalAltResult>> {
        let b = &mut *builder.borrow_mut();

        if let Some(obj) = mem::take(b) {
            Ok(obj.build())
        } else {
            Err("Builder is already consumed".into())
        }
    }
}
```


Usage
-----

It is easy to see that the Rhai script API mirrors the Rust API almost perfectly.

```rust
┌──────┐
│ Rust │
└──────┘

let mut engine = Engine::new();

engine.register_static_module("Foo", exported_module!(foo_builder).into());

let foo = FooBuilder::new().with_foo(42).with_bar(true).with_baz("Hello").build();


┌─────────────┐
│ Rhai script │
└─────────────┘

let foo = Foo::default().with_foo(42).with_bar(true).with_baz("Hello").create();
```
