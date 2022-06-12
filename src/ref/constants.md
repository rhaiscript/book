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


Automatic Global Module
-----------------------

When a [constant](constants.md) is declared at global scope, it is added to a special
[module](modules/index.md) called `global`.

[Functions](functions.md) can access those [constants](constants.md) via the special `global`
[module](modules/index.md).

```rust
const CONSTANT = 42;        // this constant is automatically added to 'global'

{
    const INNER = 0;        // this constant is not at global level
}                           // <- it goes away here

fn foo(x) {
    x *= global::CONSTANT;  // ok! 'CONSTANT' exists in 'global'

    x * global::INNER       // <- error: constant 'INNER' not found in 'global'
}
```
