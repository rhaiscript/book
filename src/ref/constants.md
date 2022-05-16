Constants
=========

Constants can be defined using the `const` keyword and are immutable.

```rust
const X;            // 'X' is a constant '()'

const X = 40 + 2;   // 'X' is a constant 42

print(X * 2);       // prints 84

X = 123;            // <- syntax error: constant modified
```

```admonish tip.small "Tip: Naming"

Constants follow the same naming rules as [variables](variables.md),
but as a convention are often named with all-capital letters.
```
