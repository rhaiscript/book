`rhai-url`: Working with Urls
=============================

{{#include ../links.md}}


`rhai-url` is an independent Rhai [package] that enables working with Urls via the
[`url`](https://crates.io/crates/url) crate.

```admonish info.side "Documentation"

See <https://docs.rs/rhai-url> for the list of functions.
```

> On `crates.io`: [`rhai-url`](https://crates.io/crates/rhai-url)
>
> On `GitHub`: [`rhaiscript/rhai-url`](https://github.com/rhaiscript/rhai-url)
>
> Package name: `FilesystemPackage`


Dependency
----------

`Cargo.toml`:

```toml
[dependencies]
rhai = "{{version}}"
rhai-url = "0.0.1"       # use rhai-url crate
```


Load Package into [`Engine`]
----------------------------

```rust
use rhai::Engine;
use rhai::packages::Package;    // needed for 'Package' trait
use rhai_url::UrlPackage;

let mut engine = Engine::new();

// Create new 'UrlPackage' instance
let url = UrlPackage::new();

// Load the package into the `Engine`
url.register_into_engine(&mut engine);
```


Example
-------

```rust
let url = Url("http://example.com/?q=query");

print(url);                 // prints 'http://example.com/?q=query'
print(url.href);            // prints 'http://example.com/?q=query'

print(url.query);           // prints 'q=query'

// fragment and hash are aliases
print(url.fragment);        // prints ''
print(url.hash);            // prints ''

url.query_clear();

print(url.query);           // prints ''

url.query_remove("q");
url.query_append("q", "name");

print(url);                 // prints 'http://example.com/?q=name'
```
