`rhai-autodocs`: Generate API Documentation
===========================================

{{#include ../links.md}}


`rhai-autodocs` helps generate API documentation, in [MarkDown] or [MDX] format, for functions
registered inside an [`Engine`] instance.

It is typically imported as a build dependency into the build script.

The [MarkDown]/[MDX] files can then be used to create a static documentation site using generators
such as [`mdbook`](https://crates.io/crates/mdbook) or [Docusaurus](https://docusaurus.io).


> On `crates.io`: [`rhai-autodocs`](https://crates.io/crates/rhai-autodocs)
>
> On `GitHub`: [`rhaiscript/rhai-autodocs`](https://github.com/ltabis/rhai-autodocs)


Usage
-----

`Cargo.toml`:

```toml
[dev-dependencies]
rhai = "{{version}}"
rhai-autodocs = "0.4"           # use rhai-autodocs crate
```

`build.rs`:

```rust
fn main() {
    // Specify an environment variable that points to the directory
    // where the documentation will be generated.
    if let Ok(docs_path) = std::env::var("DOCS_DIR") {
        let mut engine = rhai::Engine::new();

        // register custom functions and types...
        // or load packages...

        let docs = rhai_autodocs::options()
            .include_standard_packages(false)
            .generate(&engine)
            .expect("failed to generate documentation");

        // Write the documentation to a file etc.
    }
}
```
