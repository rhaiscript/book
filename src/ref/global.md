Automatic Global Module
=======================


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
