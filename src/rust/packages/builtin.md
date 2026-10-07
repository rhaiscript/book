Built-In Packages
=================

{{#include ../../links.md}}

`Engine::new` creates an [`Engine`] with the `StandardPackage` loaded.

`Engine::new_raw` creates an [`Engine`] with _no_ [package] loaded.

| Package                | Description                                                                                                 | `Core` | `Standard` | [_Lazy-loading_][lazy] |
| ---------------------- | ----------------------------------------------------------------------------------------------------------- | :----: | :--------: | :--------------------: |
| `LanguageCorePackage`  | core functions for the Rhai language                                                                        |  yes   |    yes     |          yes           |
| `ArithmeticPackage`    | arithmetic operators (e.g. `+`, `-`, `*`, `/`) for numeric types that are not built in (e.g. `u16`)         |  yes   |    yes     |          yes           |
| `BitFieldPackage`      | basic [bit-field] functions                                                                                 | **no** |    yes     |          yes           |
| `BasicIteratorPackage` | numeric ranges (e.g. `range(1, 100, 5)`), iterators for [arrays], [strings], [bit-fields] and [object maps] |  yes   |    yes     |          yes           |
| `LogicPackage`         | logical and comparison operators (e.g. `==`, `>`) for numeric types that are not built in (e.g. `u16`)      | **no** |    yes     |          yes           |
| `BasicStringPackage`   | basic string functions (e.g. `print`, `debug`, `len`) that are not built in                                 |  yes   |    yes     |          yes           |
| `BasicTimePackage`     | basic time functions (e.g. [timestamps], not available under [`no_time`] or [`no_std`])                     | **no** |    yes     |          yes           |
| `MoreStringPackage`    | additional string functions, including converting common types to string                                    | **no** |    yes     |          yes           |
| `BasicMathPackage`     | basic math functions (e.g. `sin`, `sqrt`)                                                                   | **no** |    yes     |          yes           |
| `BasicArrayPackage`    | basic [array] functions (not available under [`no_index`])                                                  | **no** |    yes     |          yes           |
| `BasicBlobPackage`     | basic [BLOB] functions (not available under [`no_index`])                                                   | **no** |    yes     |          yes           |
| `BasicMapPackage`      | basic [object map] functions (not available under [`no_object`])                                            | **no** |    yes     |          yes           |
| `BasicFnPackage`       | basic methods for [function pointers]                                                                       |  yes   |    yes     |          yes           |
| `DebuggingPackage`     | basic functions for [debugging][debugger] (requires [`debugging`])                                          |  yes   |    yes     |          yes           |
| `CorePackage`          | basic essentials                                                                                            |  yes   |    yes     |          yes           |
| `StandardPackage`      | standard library (default for `Engine::new`)                                                                | **no** |    yes     |          yes           |


`CorePackage`
-------------

If only minimal functionalities are required, register the `CorePackage` instead.

```rust
use rhai::Engine;
use rhai::packages::{Package, CorePackage};

let mut engine = Engine::new_raw();
let package = CorePackage::new();

// Register the package into the 'Engine'.
package.register_into_engine(&mut engine);
```


Lazy-Loading
------------

Use the _PackageName_`::new_lazy()` on a package built from a module with [_lazy-loading_][lazy]
enabled (via `#[export_module(manifest)]`).

[Lazy-loaded][lazy] functions are only loaded into the [`Engine`] when they are first called, which
helps conserve runtime memory usage.
