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

Implementation
--------------

```admonish tip.small "Tip: Fluent API"

This same pattern can be used to implement any [_fluent API_](https://en.wikipedia.org/wiki/Fluent_interface).
```

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
    baz: String
}

impl FooBuilder {
    pub fn new() -> Self {
        FooBuilder { foo: 0, bar: false, baz: String::new() }
    }
    /// Sets the `foo` option.
    pub fn with_foo(mut self, foo: i64) -> Self {
        self.foo = foo; self
    }
    /// Sets the `bar` option.
    pub fn with_bar(mut self, bar: bool) -> Self {
        self.bar = bar; self
    }
    /// Sets the `baz` option.
    pub fn with_baz(mut self, baz: &str) -> Self {
        self.baz = baz.to_string(); self
    }
    /// Builds the `Foo` instance.
    pub fn build(self) -> Foo {
        Foo { foo: self.foo, bar: self.bar, baz: self.baz.clone() }
    }
}

/// Builder for `Foo`.
#[export_module]
pub mod foo_builder {
    use super::{Foo, FooBuilder};
    use std::rc::Rc;
    use std::cell::RefCell;
    
    /// The builder for `Foo`.
    // This type is `Clone`.
    pub type SharedFooBuilder = Rc<RefCell<FooBuilder>>;

    /// Creates a new builder for `Foo`.
    pub fn new() -> SharedFooBuilder {
        Rc::new(RefCell::new(FooBuilder::new()))
    }
    /// Sets the `foo` option.
    #[rhai_fn(global, pure)]
    pub fn with_foo(builder: &mut SharedFooBuilder, foo: i64) -> SharedFooBuilder {
        builder.borrow_mut().with_foo(foo);
        builder.clone()
    }
    /// Sets the `bar` option.
    #[rhai_fn(global, pure)]
    pub fn with_bar(builder: &mut SharedFooBuilder, bar: bool) -> SharedFooBuilder {
        builder.borrow_mut().with_bar(bar);
        builder.clone()
    }
    /// Sets the `baz` option.
    #[rhai_fn(global, pure)]
    pub fn with_baz(builder: &mut SharedFooBuilder, baz: &str) -> SharedFooBuilder {
        builder.borrow_mut().with_baz(baz);
        builder.clone()
    }
    /// Builds the `Foo` instance.
    #[rhai_fn(global, pure)]
    pub fn build(builder: &mut SharedFooBuilder) -> Foo {
        builder.borrow().clone().build()
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

let foo = FooBuilder::new().with_foo(42).with_bar(true).with_baz("Hello, world!").build();


┌─────────────┐
│ Rhai script │
└─────────────┘

// Assuming that the `foo_builder` module is exported as `FooBuilder`...

let foo = FooBuilder::new().with_foo(42).with_bar(true).with_baz("Hello, world!").build();
```
