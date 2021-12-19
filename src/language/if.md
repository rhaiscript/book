`if` Statement
==============

{{#include ../links.md}}

`if` statements follow C syntax:

```rust no_run
if foo(x) {
    print("It's true!");
} else if bar == baz {
    print("It's true again!");
} else if baz.is_foo() {
    print("Yet again true.");
} else if foo(bar - baz) {
    print("True again... this is getting boring.");
} else {
    print("It's finally false!");
}
```

Braces Are Mandatory
--------------------

Unlike C, the condition expression does _not_ need to be enclosed in parentheses `(` .. `)`, but all
branches of the `if` statement must be enclosed within braces `{` .. `}`, even when there is only
one statement inside the branch.

Like Rust, there is no ambiguity regarding which `if` clause a branch belongs to.

```rust no_run
// Rhai is not C!
if (decision) print("I've decided!");
//            ^ syntax error, expecting '{' in statement block
```


`if`-Expressions
---------------

Like Rust, `if` statements can also be used as _expressions_, replacing the `? :` conditional
operators in other C-like languages.

```rust no_run
// The following is equivalent to C: int x = 1 + (decision ? 42 : 123) / 2;
let x = 1 + if decision { 42 } else { 123 } / 2;
x == 22;

let x = if decision { 42 }; // no else branch defaults to '()'
x == ();
```

`if`-expressions can be disabled via [`Engine::set_allow_if_expression`][options].
