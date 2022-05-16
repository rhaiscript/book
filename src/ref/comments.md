Comments
========

Comments are C-style, including `/*` ... `*/` pairs for block comments and `//` for comments to the
end of the line.

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


Doc-Comments
============

Comments starting with `///` (three slashes) or `/**` (two asterisks) are _doc-comments_.

Doc-comments can only appear in front of [function](functions.md) definitions, not any other elements.

```rust
/// This is a valid one-line doc-comment
fn foo() {}

/** This is a
 ** valid block
 ** doc-comment
 **/
fn bar(x) {
   /// Syntax error: this doc-comment is invalid
   x + 1
}

/** Syntax error: this doc-comment is invalid */
let x = 42;

/// Syntax error: this doc-comment is also invalid
{
   let x = 42;
}
```


~~~admonish tip "Tip: Special cases"

Long streams of `//////`... and `/*****`... do _NOT_ form doc-comments.
This is consistent with popular comment block styles for C-like languages.

```rust
///////////////////////////////  <- this is not a doc-comment
// This is not a doc-comment //  <- this is not a doc-comment
///////////////////////////////  <- this is not a doc-comment

// However, watch out for comment lines starting with '///'

//////////////////////////////////////////  <- this is not a doc-comment
/// This, however, IS a doc-comment!!! ///  <- doc-comment!
//////////////////////////////////////////  <- this is not a doc-comment

/****************************************
 *                                      *
 * This is also not a doc-comment block *
 * so we don't have to put this in      *
 * front of a function.                 *
 *                                      *
 ****************************************/
```
~~~
