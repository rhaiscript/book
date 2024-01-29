Auto-Generate API for Custom Type
=================================

{{#include ../links.md}}


To register a type and its API for use with an [`Engine`], the simplest method is via the [`CustomType`] trait.

A custom derive macro is provided to auto-implement [`CustomType`] on any `struct` type,
which exposes all the type's fields to an [`Engine`] all at once.

It is as simple as adding `#[derive(CustomType)]` to the type definition.

```rust
use rhai::{CustomType, TypeBuilder};    // <- necessary imports

#[derive(Clone, CustomType)]            // <- auto-implement 'CustomType'
pub struct Vec3 {                       //    for normal structs
    pub x: i64,
    pub y: i64,
    pub z: i64,
}

#[derive(Clone, CustomType)]            // <- auto-implement 'CustomType'
pub struct ABC(i64, bool, String);      //    for tuple structs

let mut engine = Engine::new();

// Register the custom types!
engine.build_type::<Vec3>()
      .build_type::<ABC>();
```


Custom Attribute Options
------------------------

The `rhai_type` attribute, with options, can be added to the fields of the type to customize the auto-generated API.

|   Option   | Applies to  |       Value       | Description                                                                                                              |
| :--------: | :---------: | :---------------: | ------------------------------------------------------------------------------------------------------------------------ |
|   `name`   | type, field | string expression | use this name instead of the type/field name.                                                                            |
|   `skip`   |    field    |      _none_       | skip this field; cannot be used with any other attribute.                                                                |
| `readonly` |    field    |      _none_       | only auto-generate getter, no setter; cannot be used with `set`.                                                         |
|   `get`    |    field    |   function path   | use this getter function (with `&self`) instead of the auto-generated getter; if `get_mut` is also set, this is ignored. |
| `get_mut`  |    field    |   function path   | use this getter function (with `&mut self`) instead of the auto-generated getter.                                        |
|   `set`    |    field    |   function path   | use this setter function instead of the auto-generated setter; cannot be used with `readonly`.                           |
|  `extra`   |    type     |   function path   | call this function after building the type to add additional API's                                                       |

### Function signatures

The signature of the function for `get` is:

> ```rust
> Fn(&T) -> V
> ```

The signature of the function for `get_mut` is:

> ```rust
> Fn(&mut T) -> V
> ```

The signature of the function for `set` is:

> ```rust
> Fn(&mut T, V)
> ```

The signature of the function for `extra` is:

> ```rust
> Fn(&mut TypeBuilder<T>)
> ```

### Example

```rust
use rhai::{CustomType, TypeBuilder};    // <- necessary imports

#[derive(Debug, Clone)]
#[derive(CustomType)]                   // <- auto-implement 'CustomType'
pub struct ABC(
    #[rhai_type(skip)]                  // <- 'field0' not included
    i64,

    #[rhai_type(readonly)]              // <- only auto getter, no setter for 'field1'
    i64,
    
    #[rhai_type(name = "flag")]         // <- override property name for 'field2'
    bool,
    
    String                              // <- auto getter/setter for 'field3'
);

#[derive(Default, Clone)]
#[derive(CustomType)]                   // <- auto-implement 'CustomType'
#[rhai_type(name = "MyFoo", extra = Self::build_extra)] // <- call this type 'MyFoo' and use 'build_extra' to add additional API's
pub struct Foo {
    #[rhai_type(skip)]                  // <- field not included
    dummy: i64,

    #[rhai_type(readonly)]              // <- only auto getter, no setter for 'bar'
    bar: i64,

    #[rhai_type(name = "flag")]         // <- override property name
    baz: bool,                          // <- auto getter/setter for 'baz'

    #[rhai_type(get = Self::qux)]       // <- call custom getter (with '&self') for 'qux'
    qux: char,                          // <- auto setter for 'qux'

    #[rhai_type(set = Self::set_hello)] // <- call custom setter for 'hello'
    hello: String                       // <- auto getter for 'hello'
}

impl Foo {
    /// Regular field getter function with `&self`
    pub fn qux(&self) -> char {
        self.qux
    }

    /// Special setter implementation for `hello`
    pub fn set_hello(&mut self, value: String) {
        self.hello = if self.baz {
            let mut s = self.hello.clone();
            s.push_str(&value);
            for _ in 0..self.bar { s.push('!'); }
            s
        } else {
            value
        };
    }

    /// Additional API's
    fn build_extra(builder: &mut TypeBuilder<Self>) {
        // Register constructor function
        builder.with_fn("new_foo", || Self::default());
    }
}

#[derive(Debug, Clone, Eq, PartialEq, CustomType)]
#[rhai_fn(extra = vec3_build_extra)]
pub struct Vec3 {
    #[rhai_type(get = Self::x, set = Self::set_x)]
    x: i64,
    #[rhai_type(get = Self::y, set = Self::set_y)]
    y: i64,
    #[rhai_type(get = Self::z, set = Self::set_z)]
    z: i64,
}

impl Vec3 {
    fn new(x: i64, y: i64, z: i64) -> Self { Self { x, y, z } }
    fn x(&self) -> i64 { self.x }
    fn set_x(&mut self, x: i64) { self.x = x }
    fn y(&self) -> i64 { self.y }
    fn set_y(&mut self, y: i64) { self.y = y }
    fn z(&self) -> i64 { self.z }
    fn set_z(&mut self, z: i64) { self.z = z }
}

fn vec3_build_extra(builder: &mut TypeBuilder<Self>) {
    // Register constructor function
    builder.with_fn("Vec3", Self::new);
}
```

The above is equivalent to the following manual implementations of the [`CustomType`] trait.

```rust
impl CustomType for ABC {
    fn build(mut builder: TypeBuilder<Self>)
    {
        builder.with_name("ABC");
        builder.with_get("field1", |obj: &mut Self| obj.1.clone());
        builder.with_get_set("flag",
            |obj: &mut Self| obj.2.clone(),
            |obj: &mut Self, val| obj.2 = val
        );
        builder.with_get_set("field3",
            |obj: &mut Self| obj.3.clone(),
            |obj: &mut Self, val| obj.3 = val
        );
    }
}

impl CustomType for Foo {
    fn build(mut builder: TypeBuilder<Self>)
    {
        builder.with_name("MyFoo");
        builder.with_get("bar", |obj: &mut Self| obj.bar.clone());
        builder.with_get_set("flag",
            |obj: &mut Self| obj.baz.clone(),
            |obj: &mut Self, val| obj.baz = val
        );
        builder.with_get_set("qux",
            |obj: &Self| Self::qux(&*obj)),
            |obj: &mut Self, val| obj.qux = val
        )
        builder.with_get_set("hello",
            |obj: &mut Self| obj.hello.clone(),
            Self::set_hello
        );
        Self::build_extra(&mut builder);
    }
}

impl CustomType for Vec3 {
    fn build(mut builder: TypeBuilder<Self>)
    {
        builder.with_name("Vec3");
        builder.with_get_set("x", |obj: &mut Self| Self::x(&*obj), Self::set_x);
        builder.with_get_set("y", |obj: &mut Self| Self::y(&*obj), Self::set_y);
        builder.with_get_set("z", |obj: &mut Self| Self::z(&*obj), Self::set_z);
        vec3_build_extra(&mut builder);
    }
}
```
