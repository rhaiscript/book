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

| Flag/Option       |    Parameter    |    Default    | Description                                                     |
| ----------------- | :-------------: | :-----------: | --------------------------------------------------------------- |
| `-h`, `--help`    |                 |               | print help                                                      |
| `-V`, `--version` |                 |               | print version                                                   |
| `--verbose`       |                 |               | print diagnostic messages                                       |
| `--config`        |   _\<file\>_    | `./rhai.toml` | set configuration file                                          |
| `-D`, `--dest`    | _\<directory\>_ |   `./dist`    | set destination path for documentation output                   |
| `-d`, `--dir`     | _\<directory\>_ |      `.`      | set source path for Rhai scripts                                |
| `-p`, `--pages`   | _\<directory\>_ |   `./pages`   | set source path for additional [MarkDown] page files to include |


`rhai.toml`
-----------

The file `rhai.toml` contains configuration options for `rhai-doc` and must be placed in the source directory.

Example:

```toml
name = "My Rhai Project"                # project name
color = [246, 119, 2]                   # theme color
index = "home.md"                       # this file becomes 'index.html`
root = "https://example.com/docs/"      # root URL for generated site
icon = "logo.svg"                       # project icon
stylesheet = "my_stylesheet.css"        # custom stylesheet
extension = "rhai"                      # script extension
google_analytics = "G-ABCDEF1234"       # Google Analytics ID

[[links]]                               # external link for 'Blog'
name = "Blog"
link = "https://example.com/blog"

[[links]]                               # external link for 'Tools'
name = "Tools"
link = "https://example.com/tools"
```

Options for `rhai.toml`:

|       Option       | Optional |          Value           | Description                                                   |
| :----------------: | :------: | :----------------------: | ------------------------------------------------------------- |
|       `name`       |   yes    |          string          | name of project &ndash; used as titles on documentation pages |
|      `color`       |   yes    | RGB values (0-255) array | theme color for generated documentation                       |
|      `index`       |    no    |        file name         | main [MarkDown] file &ndash; becomes `index.html`             |
|       `root`       |   yes    |        URL string        | root URL generated as part of documentation                   |
|       `icon`       |   yes    |        file path         | project icon                                                  |
|    `stylesheet`    |   yes    |        file path         | custom stylesheet                                             |
|    `extension`     |   yes    |     extension string     | script file extension (default `rhai`)                        |
| `google_analytics` |   yes    |        ID string         | [Google Analytics](https://analytics.google.com) ID           |
|    `[[links]]`     |   yes    |          table           | external links                                                |
|   &bull; `name`    |    no    |          string          | &bull; title of external link                                 |
|   &bull; `link`    |    no    |        URL string        | &bull; URL of external link                                   |
