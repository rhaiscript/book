Comments
========

{{#include ../links.md}}

Comments are C-style, including '`/*` ... `*/`' pairs for block comments
and '`//`' for comments to the end of the line.

Comments can be nested.

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
