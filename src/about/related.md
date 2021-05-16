Related Resources
=================

{{#include ../links.md}}


Online Resources for Rhai
-------------------------

* [GitHub](https://github.com/rhaiscript) &ndash; Rhai organization

* [`rhai.rs`](https://rhai.rs) &ndash; Home website

* [`crates.io`](https://crates.io/crates/rhai) &ndash; Rhai crate

* [`DOCS.RS`](https://docs.rs/rhai) &ndash; Rhai API documentation

* [`LIB.RS`](https://lib.rs/crates/rhai) &ndash; Rhai library info

* [Discord Chat](https://discord.gg/HquqbYFcZ9) &ndash; Rhai channel

* [Reddit](https://www.reddit.com/r/Rhai) &ndash; Rhai community


External Tools
--------------

* [Online Playground][playground] &ndash; Run Rhai scripts directly from an editor in the browser

* [`rhai-doc`] &ndash; Rhai script documentation tool


Syntax Highlighting
-------------------

* [VS Code Extension](https://marketplace.visualstudio.com/items?itemName=rhaiscript.vscode-rhai) &ndash;
  Support `.rhai` script files syntax highlighting for Visual Studio Code

* [Sublime Text 3 Plugin](https://packagecontrol.io/packages/Rhai) &ndash;
  Support `.rhai` script files syntax highlighting for Sublime Text 3

* For other syntax highlighting purposes, e.g. `vim`, `highlight.js`, both Rust or JavaScript can be used successfully.
  
  Use `rust` when there is no [string interpolation][string]. This way, [closures] and [functions] (via
  the `fn` keyword) are styled properly. Elements not highlighted include:
  * [strings interpolation][string]
  * the `switch`, [`import`] and [`export`] statements
  * the `this` and [`private`] keywords
  * built-in functions such as `Fn`, `call`, [`type_of`], `is_shared`, `is_def_var`, `is_def_fn`

  Use `js` (JavaScript) when there is [strings interpolation][string].  Elements not highlighted include:
  * [functions] definition (via the `fn` keyword)
  * [closures] (via the Rust-like `|...| { ... }` syntax)
  * built-in functions such as `Fn`, `call`, [`type_of`], `is_shared`, `is_def_var`, `is_def_fn`


Other Cool Projects
-------------------

* [ChaiScript](http://chaiscript.com) &ndash;
  A strong inspiration for Rhai.  An embedded scripting language for C++.

* Check out the list of [scripting languages for Rust](https://github.com/rust-unofficial/awesome-rust#scripting)
  on [awesome-rust](https://github.com/rust-unofficial/awesome-rust)
