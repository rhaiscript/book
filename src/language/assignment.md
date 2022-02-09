Assignments
===========

{{#include ../links.md}}


Value assignments to [variables] use the `=` symbol.


```rust,no_run
let foo = 42;

bar = 123 * 456 - 789;

x[1][2].prop = do_calculation();
```

The left-hand-side (LHS) of an assignment statement must be a valid
_[l-value](https://en.wikipedia.org/wiki/Value_(computer_science))_, which must be rooted in a
[variable], potentially extended via indexing or properties.

Expressions that are not valid _l-values_ cannot be assigned to.

```rust,no_run
x = 42;                 // variable is an l-value

x[1][2][3] = 42         // variable indexing is an l-value

x.prop1.prop2 = 42;     // variable property is an l-value

foo(x) = 42;            // syntax error: function call is not an l-value

x.foo() = 42;           // syntax error: method call is not an l-value

(x + y) = 42;           // syntax error: binary expression is not an l-value
```
