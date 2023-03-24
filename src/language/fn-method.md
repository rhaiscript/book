`this` &ndash; Simulating an Object Method
==========================================

{{#include ../links.md}}

```admonish warning.side "Functions are pure"

The only way for a script-defined [function] to change an external value is via `this`.
```

Arguments passed to script-defined [functions] are always by _value_ because [functions] are _pure_.

However, [functions] can also be called in _method-call_ style (not available under [`no_object`]):

> _object_ `.` _method_ `(` _parameters_ ... `)`

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


Elvis Operator
--------------

The [_Elvis_ operator][elvis] can be used to short-circuit the method call when the object itself is [`()`].

> _object_ `?.` _method_ `(` _parameters_ ... `)`

In the above, the _method_ is never called if _object_ is [`()`].


Bind to `this` for Module Functions
-----------------------------------

The _method-call_ syntax is not possible for [functions] [imported][`import`] from [modules].

```js
import "my_module" as foo;

let x = 42;

x.foo::change_value(1);     // <- syntax error
```

In order to call a [module] [function] as a method, it must be defined with a restriction on the
type of object pointed to by `this`:

```js
┌────────────────┐
│ my_module.rhai │
└────────────────┘

// This is a method call requiring 'this' to be an integer.
// Methods are automatically marked global when importing this module.
fn int.change_value_impl(offset) {
    // 'this' is guaranteed to be an integer
    this += offset;
}


┌───────────┐
│ main.rhai │
└───────────┘

import "my_module";

let x = 42;

x.change_value(1);

x == 43;
```
