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

* Install [`mdbook-tera`](https://github.com/avitex/mdbook-tera) (for templating)

```sh
cargo install mdbook-tera
```

* Run `build`

```sh
mdbook build
```


Configuration Settings
----------------------

Settings are stored in `src/context.json`:

| Setting    | Description                                                                             |
| ---------- | --------------------------------------------------------------------------------------- |
| `version`  | version of Rhai                                                                         |
| `repoHome` | points to the [root of the GitHub repo](https://github.com/rhaiscript/rhai/blob/master) |
| `rootUrl`  | sub-directory for _The Book_, e.g. `/book`                                              |
