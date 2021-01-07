The Rhai Book
=============

[_The Rhai Book_](https://rhaiscript.github.io/rhai) serves as Rhai's primary
documentation and tutorial resource.


How to Build from Source
------------------------

* Install [`mdbook`](https://github.com/rust-lang/mdBook):

```bash
cargo install mdbook
```

* Install [`mdbook-tera`](https://github.com/avitex/mdbook-tera) (for templating):

```bash
cargo install mdbook-tera
```

* Run build in source directory:

```bash
mdbook build
```


Configuration Settings
----------------------

Settings stored in `context.json`:

| Setting    | Description                                                                                  |
| ---------- | -------------------------------------------------------------------------------------------- |
| `version`  | version of Rhai                                                                              |
| `repoHome` | points to the [root of the GitHub repo](https://github.com/rhaiscript/rhai/blob/master)      |
| `repoTree` | points to the [root of the GitHub repo tree](https://github.com/rhaiscript/rhai/tree/master) |
| `rootUrl`  | sub-directory for the root domain, e.g. `/rhai`                                              |
