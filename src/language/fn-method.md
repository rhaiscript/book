`this` &ndash; Simulating an Object Method
==========================================

{{#include ../links.md}}

```admonish warning.side.wide "Functions are pure"

The only way for a script-defined [function] to change an external value is via `this`.
```

Arguments passed to script-defined [functions] are always by _value_ because [functions] are _pure_.

However, script-defined [functions] can also be called in _method-call_ style:

> _object_ `.` _function_ `(` _parameters_ ... `)`

When a [function] is called this way, the keyword `this` binds to the object in the method call and
can be changed.

```rust
fn change() {       // note that the method does not need a parameter
    this = 42;      // 'this' binds to the object in method-call
}

let x = 500;

x.change();         // call 'change' in method-call style, 'this' binds to 'x'

x == 42;            // 'x' is changed!

change();           // <- error: 'this' is unbound
```
