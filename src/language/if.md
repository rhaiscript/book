If Statement
============

{{#include ../links.md}}

`if` statements follow C syntax.

```rust,no_run
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

~~~admonish warning.small "Braces are mandatory"

Unlike C, the condition expression does _not_ need to be enclosed in parentheses `(`...`)`, but all
branches of the `if` statement must be enclosed within braces `{`...`}`, even when there is only
one statement inside the branch.

Like Rust, there is no ambiguity regarding which `if` clause a branch belongs to.

```rust,no_run
// Rhai is not C!
if (decision) print(42);
//            ^ syntax error, expecting '{'
```
~~~
