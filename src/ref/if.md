If Statement
============

`if` statements follow C syntax.

```rust
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

There is no ambiguity regarding which `if` clause a branch belongs to.

```rust
// Rhai is not C!
if (decision) print(42);
//            ^ syntax error, expecting '{'
```
~~~


If Expression
=============

`if` statements can also be used as _expressions_, replacing the `? :` conditional operators in
other C-like languages.

```rust
// The following is equivalent to C: int x = 1 + (decision ? 42 : 123) / 2;
let x = 1 + if decision { 42 } else { 123 } / 2;
x == 22;

let x = if decision { 42 }; // no else branch defaults to '()'
x == ();
```

~~~admonish danger.small "Statement before expression"

Beware that, like Rust, `if` is parsed primarily as a statement where it makes sense.
This is to avoid surprises.

```rust
fn index_of(x) {
    // 'if' is parsed primarily as a statement
    if this.contains(x) {
        return this.find_index(x)
    }

    -1
}
```

The above will not be parsed as a single expression:

```rust
fn index_of(x) {
    if this.contains(x) { return this.find_index(x) } - 1
    //                          error due to '() - 1' ^
}

```

To force parsing as an expression, parentheses are required:

```rust
fn calc_index(b, offset) {
    (if b { 1 } else { 0 }) + offset
//  ^---------------------^ parentheses
}
```
~~~
