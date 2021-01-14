Rhai Script Documentation Tool
=============================

{{#include ../links.md}}


The Rhai Script Documentation Tool, `rhai-doc`, takes a source directory and scans for
Rhai script files (recursively), building a web-based documentation site for all functions
defined.  Documentation is taken from [MarkDown] [doc-comments] on the functions.


Author: [`@semirix`](https://github.com/semirix)

Repo: [On GitHub](https://github.com/rhaiscript/rhai-doc)

Binary: `rhai-doc`


Flags and Options
-----------------

|    Flag/Option    |    Parameter    | Description                                                                               |
| :---------------: | :-------------: | ----------------------------------------------------------------------------------------- |
|  `-h`, `--help`   |                 | print help                                                                                |
| `-V`, `--version` |                 | print version                                                                             |
|  `-D`, `--dest`   | _\<directory\>_ | set destination path for documentation output (default `./dist`)                          |
|   `-d`, `--dir`   | _\<directory\>_ | set source path for Rhai script files (default `.`)                                       |
|  `-p`, `--pages`  | _\<directory\>_ | set source path for additional [MarkDown] files to include in documentation (default `.`) |


`rhai.toml`
-----------

The file `rhai.toml` contains configuration options for `rhai-doc` and must be placed in the source
directory.

Example:

```toml
name = "My Rhai Project"                # project name
colour = [246, 119, 2]                  # theme color
index = "home.md"                       # this file becomes 'index.html`
root = "https://example.com/docs/"      # root URL for generated site
icon = "logo.svg"                       # project icon
extension = "rhai"                      # script extension

[[links]]                               # external link for 'Blog'
name = "Blog"
link = "https://example.com/blog"

[[links]]                               # external link for 'Tools'
name = "Tools"
link = "https://example.com/tools"
```

Options for `rhai.toml`:

|    Option    |      Value       | Description                                                   |
| :----------: | :--------------: | ------------------------------------------------------------- |
|    `name`    |      string      | name of project &ndash; used as titles on documentation pages |
|   `colour`   | RGB value string | theme color for generated documentation                       |
|   `index`    |    file name     | main MarkDown file &ndash; becomes `index.html`               |
|    `root`    |    URL string    | root URL generated as part of documentation                   |
|    `icon`    |    file path     | project icon                                                  |
| `extension`  | extension string | script file extension (default `rhai`)                        |
| `[[links]]`  |      table       | external links                                                |
| `links.name` |      string      | &bull; title of external link                                 |
| `links.link` |    URL string    | &bull; URL of external link                                   |
