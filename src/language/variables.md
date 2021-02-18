Variables
=========

{{#include ../links.md}}

Valid Names
-----------

Variables in Rhai follow normal C naming rules &ndash; must contain only ASCII letters, digits and underscores `_`,
and cannot start with a digit.

For example: `_c3po` and `r2d2` are valid variable names, but `3abc` is not.

However, unlike Rust, a variable name must also contain at least one ASCII letter, and an ASCII letter must come before any digit.
In other words, the first character that is not an underscore `_` must be an ASCII letter and not a digit.

Therefore, some names acceptable to Rust, like `_`, `_42foo`, `_1` etc., are not valid in Rhai.
This restriction is to reduce confusion because, for instance, `_1` can easily be misread (or mis-typed) as `-1`.

Variable names are case _sensitive_.

Variable names also cannot be the same as a [keyword].

### Unicode Standard Annex #31 Identifiers

The [`unicode-xid-ident`] feature expands the allowed characters for variable names to the set defined by
[Unicode Standard Annex #31](http://www.unicode.org/reports/tr31/).


Declare a Variable
------------------

Variables are declared using the `let` keyword.

Variables do not have to be given an initial value.
If none is provided, it defaults to [`()`].

A variable defined within a statement block is _local_ to that block.

Use `is_def_var` to detect if a variable is defined.

```rust,no_run
let x;              // ok - value is '()'
let x = 3;          // ok
let _x = 42;        // ok
let x_ = 42;        // also ok
let _x_ = 42;       // still ok

let _ = 123;        // <- syntax error: illegal variable name
let _9 = 9;         // <- syntax error: illegal variable name

let x = 42;         // variable is 'x', lower case
let X = 123;        // variable is 'X', upper case
x == 42;
X == 123;

{
    let x = 999;    // local variable 'x' shadows the 'x' in parent block
    x == 999;       // access to local 'x'
}
x == 42;            // the parent block's 'x' is not changed

is_def_var("x") == true;

is_def_var("_x") == true;

is_def_var("y") == false;
```
