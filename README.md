The Rhai Book
=============

[_The Rhai Book_](https://rhai.rs/book) serves as Rhai's primary
documentation and tutorial resource.


How to Build from Source
------------------------

* Install [`mdbook`](https://github.com/rust-lang/mdBook)

```sh
cargo install mdbook
```

* Install [`mdbook-tera`](https://crates.io/crates/mdbook-tera) (for templating)

```sh
cargo install mdbook-tera
```

* Install [`mdbook-admonish`](https://crates.io/crates/mdbook-admonish) (for styling)

```sh
cargo install mdbook-admonish
```

* Run `build`

```sh
mdbook build
```

### Warning: Recompile when `mdbook` is updated

`mdbook-tera` and `mdbook-admonish` depend on particular versions of `mdbook`.

When `mdbook` is updated, it is best to reinstall both plugins to make sure that there are no
version conflicts.


Configuration Settings
----------------------

Settings are stored in `src/context.toml`:

| Setting    | Description                                                                             |
| ---------- | --------------------------------------------------------------------------------------- |
| `version`  | version of Rhai                                                                         |
| `repoHome` | points to the [root of the GitHub repo](https://github.com/rhaiscript/rhai/blob/master) |
| `rootUrl`  | sub-directory for _The Book_, e.g. `/book`                                              |
