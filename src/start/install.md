Install the Rhai Crate
======================

{{#include ../links.md}}

In order to use Rhai in a project, the Rhai crate must first be made a dependency.


~~~admonish info "Use specific version"

The easiest way is to install the Rhai crate from [`crates.io`](https://crates.io/crates/rhai/),
starting by looking up the latest version and adding this line under `dependencies` in the project's `Cargo.toml`:

```toml
[dependencies]
rhai = "{{version}}"    # assuming {{version}} is the latest version
```
~~~

~~~admonish note "Use latest release version"

Automatically use the latest released crate version on [`crates.io`](https://crates.io/crates/rhai/):

```toml
[dependencies]
rhai = "*"
```
~~~

~~~admonish tip "Use latest development version"

Crate versions are released on [`crates.io`](https://crates.io/crates/rhai/) infrequently,
so to track the latest features, enhancements and bug fixes, pull directly from
[GitHub](https://github.com/rhaiscript/rhai):

```toml
[dependencies]
rhai = { git = "https://github.com/rhaiscript/rhai" }
```
~~~
