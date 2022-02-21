Doc-Comments
============

{{#include ../links.md}}

Similar to Rust, [comments] starting with `///` (three slashes) or `/**` (two asterisks)
are _doc-comments_.

Doc-comments can only appear in front of [function] definitions, not any other elements.
Therefore, doc-comments are not available under [`no_function`].

~~~admonish warning "Requires `metadata`"

Doc-comments are only supported under the [`metadata`] feature.

If [`metadata`] is not active, doc-comments are treated as normal [comments].
~~~

```rust,no_run
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
This is consistent with popular [comment] block styles for C-like languages.

```rust,no_run
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


Using Doc-Comments
------------------

Doc-comments are stored within the script's [`AST`] after compilation.

The `AST::iter_functions` method provides a `ScriptFnMetadata` instance for each function defined
within the script, which includes doc-comments.

Doc-comments never affect the evaluation of a script nor do they incur significant performance overhead.
However, third party tools can take advantage of this information to auto-generate documentation for
Rhai script functions.
