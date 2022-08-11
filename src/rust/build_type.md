Register a Custom Type via the Type Builder
===========================================

{{#include ../links.md}}

Sometimes it is convenient to package a [custom type]'s API (i.e. [methods],
[properties][getters/setters], [indexers] and [type iterators]) together such that they can be more
easily managed.

This can be achieved simply by implementing the `CustomType` trait, which contains only a single method:

> `fn build(builder: TypeBuilder<T>)`

The `TypeBuilder` parameter provides a range of convenient methods to register [methods], property
[getters/setters], [indexers] and [type iterators] of a [custom type]:

| Method                    | Description                                                                 |
| ------------------------- | --------------------------------------------------------------------------- |
| `with_name`               | set a friendly name                                                         |
| `with_fn`                 | register a [method]                                                         |
| `with_result_fn`          | register a [fallible][fallible function] [method]                           |
| `with_get`                | register a property [getter][getters/setters]                               |
| `with_get_result`         | register a [fallible][fallible function] property [getter][getters/setters] |
| `with_set`                | register a property [getter][getters/setters]                               |
| `with_set_result`         | register a [fallible][fallible function] property [getter][getters/setters] |
| `with_get_set`            | register property [getters/setters]                                         |
| `with_indexer_get`        | register an [indexer] get function                                          |
| `with_indexer_get_result` | register a [fallible][fallible function] [indexer] get function             |
| `with_indexer_set`        | register an [indexer] set function                                          |
| `with_indexer_set_result` | register a [fallible][fallible function] [indexer] set function             |
| `with_indexer_get_set`    | register [indexer] get/set functions                                        |
| `is_iterable`             | register a [type iterator] if the [custom type] is iterable                 |

```admonish tip.small "Tip: Use plugin module if starting from scratch"

The `CustomType` trait is typically used on external types that are already defined.

To define a [custom type] and implement its API from scratch, it is more convenient to use a [plugin module].
```


Example
-------

```rust
// Custom type
#[derive(Debug, Clone, PartialEq)]
struct Vec3 {
    x: i64,
    y: i64,
    z: i64,
}

// Custom type API
impl Vec3 {
    fn new(x: i64, y: i64, z: i64) -> Self {
        Self { x, y, z }
    }
    fn get_x(&mut self) -> i64 {
        self.x
    }
    fn set_x(&mut self, x: i64) {
        self.x = x
    }
    fn get_y(&mut self) -> i64 {
        self.y
    }
    fn set_y(&mut self, y: i64) {
        self.y = y
    }
    fn get_z(&mut self) -> i64 {
        self.z
    }
    fn set_z(&mut self, z: i64) {
        self.z = z
    }
    fn get_component(&mut self, idx: i64) -> Result<i64, Box<EvalAltResult>> {
        match idx {
            0 => Ok(self.x),
            1 => Ok(self.y),
            2 => Ok(self.z),
            _ => Err(EvalAltResult::ErrorIndexNotFound(idx.Into(), Position::NONE).into()),
        }
    }
}

// The custom type can even be iterated!
impl IntoIterator for Vec3 {
    type Item = i64;
    type IntoIter = std::vec::IntoIter<Self::Item>;

    fn into_iter(self) -> Self::IntoIter {
        vec![self.x, self.y, self.z].into_iter()
    }
}

// Use 'CustomType' to register the entire API
impl CustomType for Vec3 {
    fn build(mut builder: TypeBuilder<Self>) {
        builder
            .with_name("Vec3")
            .with_fn("vec3", Self::new)
            .is_iterable()
            .with_get_set("x", Self::get_x, Self::set_x)
            .with_get_set("y", Self::get_y, Self::set_y)
            .with_get_set("z", Self::get_z, Self::set_z)
            .with_indexer_get_result(Self::get_component);
    }
}

let mut engine = Engine::new();

// Register the custom type in one go!
engine.build_type::<Vec3>();
```
