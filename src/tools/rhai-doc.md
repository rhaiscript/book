Rhai Script Documentation Tool
=============================

{{#include ../links.md}}


The Rhai Script Documentation Tool, `rhai-doc`, takes a source directory and scans for
Rhai script files (recursively), building a web-based documentation site for all functions
defined.  Documentation is taken from [MarkDown] [doc-comments] on the functions.


Author: [`@semirix`](https://github.com/semirix)

Repo: [On GitHub](https://github.com/rhaiscript/rhai-doc)

Binary: [On `crates.io`](https://crates.io/crates/rhai-doc)


Install
-------

```sh
cargo install rhai-doc
```


Flags and Options
-----------------

| Flag/Option       |    Parameter    |      Default      | Description                                                                            |
| ----------------- | :-------------: | :---------------: | -------------------------------------------------------------------------------------- |
| `-h`, `--help`    |                 |                   | print help                                                                             |
| `-V`, `--version` |                 |                   | print version                                                                          |
| `-a`, `--all`     |                 |                   | generate documentation for all functions, including [`private`] ones _(default false)_ |
| `-v`              |                 |                   | use multiple to set verbosity: 1=silent, 2,3 _(default)_=full                          |
| `-c`, `--config`  |   _\<file\>_    |    `rhai.toml`    | set configuration file                                                                 |
| `-D`, `--dest`    | _\<directory\>_ |      `dist`       | set destination directory for documentation output                                     |
| `-d`, `--dir`     | _\<directory\>_ | current directory | set source directory for Rhai scripts                                                  |
| `-p`, `--pages`   | _\<directory\>_ |      `pages`      | set source directory for additional [MarkDown] page files to include                   |


Commands
--------

| Command | Description                                           | Example        |
| ------- | ----------------------------------------------------- | -------------- |
| _none_  | generate documentation                                | `rhai-doc`     |
| `new`   | create a skeleton `rhai.toml` in the source directory | `rhai-doc new` |


Configuration file
------------------

A configuration file, which is usually named `rhai.toml`, contains configuration options for
`rhai-doc` and must be placed in the source directory.

A skeleton `rhai.toml` can be generated inside the source directory via the `new` command.

An alternate configuration file can be specified via the `--config` option.

### Example

```toml
name = "My Rhai Project"                # project name
color = [246, 119, 2]                   # theme color
root = "/docs/"                         # root URL for generated site
index = "home.md"                       # this file becomes 'index.html'
icon = "logo.svg"                       # project icon
stylesheet = "my_stylesheet.css"        # custom stylesheet
code_theme = "atom-one-light"           # 'highlight.js' theme
code_lang = "ts"                        # default language for code blocks
extension = "rhai"                      # script extension
google_analytics = "G-ABCDEF1234"       # Google Analytics ID

[[links]]                               # external link for 'Blog'
name = "Blog"
link = "https://example.com/blog"

[[links]]                               # external link for 'Tools'
name = "Tools"
link = "https://example.com/tools"
```

Configuration Options
---------------------

| Option             | Value type               |     Default     | Description                                                                             |
| ------------------ | ------------------------ | :-------------: | --------------------------------------------------------------------------------------- |
| `name`             | string                   |     _none_      | name of project &ndash; used as titles on documentation pages                           |
| `color`            | RGB values (0-255) array | `[246, 119, 2]` | theme color for generated documentation                                                 |
| `root`             | URL string               |     _none_      | root URL generated as part of documentation                                             |
| `index`            | file path                |     _none_      | main [MarkDown] file &ndash; becomes `index.html`                                       |
| `icon`             | file path                |    Rhai icon    | project icon                                                                            |
| `stylesheet`       | file path                |     _none_      | custom stylesheet                                                                       |
| `code_theme`       | theme string             |    `default`    | [`highlight.js`](https://highlightjs.org/) theme for syntax highlighting in code blocks |
| `code_lang`        | language string          |      `ts`       | default language for code blocks                                                        |
| `extension`        | extension string         |     `.rhai`     | script files extension (default `.rhai`)                                                |
| `google_analytics` | ID string                |     _none_      | [Google Analytics](https://analytics.google.com) ID                                     |
| `[[links]]`        | table                    |     _none_      | external links                                                                          |
| &bull; `name`      | string                   |     _none_      | &bull; title of external link                                                           |
| &bull; `link`      | URL string               |     _none_      | &bull; URL of external link                                                             |


[MarkDown] Pages
----------------

By default, `rhai-doc` will generate documentation pages from a `pages` sub-directory
under the scripts directory.

Alternatively, you can specify another location via the `--pages` option.
