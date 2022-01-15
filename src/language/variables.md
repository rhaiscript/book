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
This restriction is to reduce confusion because, for instance, `_1` can easily be misread (or mistyped) as `-1`.

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


Avoid Variable Names Longer Than 11 Characters on 32-Bit System
--------------------------------------------------------------

Internally, Rhai uses [`SmartString`] which avoids allocations unless a string is over its internal
limits (23 ASCII characters on 64-bit, but only 11 ASCII characters on 32-bit).

On 64-bit systems, _most_ variable names are shorter than 23 characters, so this is unlikely to
become an issue.

However, on 32-bit systems, take care to limit, where possible, variable names to within 11 characters.
This is particularly true for local variables inside a hot loop, where they are created and destroyed
in rapid succession.


Use Before Definition
---------------------

By default, variables do not need to be defined before they are used.
If a variable accessed by a script is not defined previously, within the same script,
it is searched for inside the [`Scope`] (if any) passed into the `Engine::eval_with_scope` call.

If no [`Scope`] is used to evaluate the script, that an undefined variable causes a runtime
error when accessed.


Strict Variables Mode
---------------------

With [`Engine::set_strict_variables`][options], it is possible to turn on
[_Strict Variables_][strict variables] mode.

When [strict variables] mode is active, accessing a variable not previously defined within
the same script directly causes a parse error when compiling the script.

Turn on [strict variables] mode if no [`Scope`] is to be provided for script evaluation runs.
This way, variable access errors are caught during compile time instead of runtime.
