Comments
========

{{#include ../links.md}}

Comments are C-style, including `/*` ... `*/` pairs for block comments and `//` for comments to the
end of the line.

Block comments can be nested.

```rust
let /* intruder comment */ name = "Bob";

// This is a very important one-line comment

/* This comment spans
   multiple lines, so it
   only makes sense that
   it is even more important */

/* Fear not, Rhai satisfies all nesting needs with nested comments:
   /*/*/*/*/**/*/*/*/*/
*/
```


Module Documentation
--------------------

Comment lines starting with `//!` make up the _module documentation_.

They are used to document the containing [module] &ndash; or for a Rhai script file,
to document the file itself.

~~~admonish warning.small "Requires `metadata`"

Module documentation is only supported under the [`metadata`] feature.

If [`metadata`] is not active, they are treated as normal [comments].
~~~

```rust
//! Documentation for this script file.
//! This script is used to calculate something and display the result.

fn calculate(x) {
   ...
}

fn display(msg) {
   //! Module documentation can be placed anywhere within the file.
   ...
}

//! All module documentation lines will be collected into a single block.
```

For the example above, the module documentation block is:

```rust
//! Documentation for this script file.
//! This script is used to calculate something and display the result.
//! Module documentation can be placed anywhere within the file.
//! All module documentation lines will be collected into a single block.
```
