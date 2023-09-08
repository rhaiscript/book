`this` &ndash; Simulating an Object Method
==========================================

```admonish warning.side "Functions are pure"

The only way for a script-defined [function](functions.md) to change an external value is via `this`.
```

Arguments passed to script-defined [functions](functions.md) are always by _value_ because
[functions](functions.md) are _pure_.

However, [functions](functions.md) can also be called in _method-call_ style:

> _object_ `.` _method_ `(` _parameters_ ... `)`

When a [function](functions.md) is called this way, the keyword `this` binds to the object in the
method call and can be changed.

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

The [_Elvis_ operator](https://en.wikipedia.org/wiki/Elvis_operator) can be used to short-circuit
the method call when the object itself is `()`.

> _object_ `?.` _method_ `(` _parameters_ ... `)`

In the above, the _method_ is never called if _object_ is `()`.


Restrict the Type of `this` in Function Definitions
---------------------------------------------------

```admonish tip.side.wide "Tip: Automatically global"

Methods defined this way are automatically exposed to the global namespace.
```

In many cases it may be desirable to implement _methods_ for different custom types using
script-defined [functions](functions.md).

### The Problem

Doing so is brittle and requires a lot of type checking code because there can only be one
[function](functions.md) definition for the same name and arity:

```js
// Really painful way to define a method called 'do_update' on various data types
fn do_update(x) {
    switch type_of(this) {
        "i64" => this *= x,
        "string" => this.len += x,
        "bool" if this => this *= x,
        "bool" => this *= 42,
        "MyType" => this.update(x),
        "Strange-Type#Name::with_!@#symbols" => this.update(x),
        _ => throw `I don't know how to handle ${type_of(this)}`!`
    }
}
```

### The Solution

With a special syntax, it is possible to restrict a [function](functions.md) to be callable only
when the object pointed to by `this` is of a certain type:

> `fn`  _type name_ `.` _method_ `(` _parameters_ ... `)  {`  ...  `}`

or in quotes if the type name is not a valid identifier itself:

> `fn`  `"`_type name string_`"` `.` _method_ `(` _parameters_ ... `)  {`  ...  `}`

~~~admonish warning.small "Type name must be the same as `type_of`"

The _type name_ specified in front of the [function](functions.md) name must match the output of
[`type_of`](type-of.md) for the required type.
~~~

~~~admonish tip.small "Tip: `int` and `float`"
`int` can be used in place of the system integer type (usually `i64` or `i32`).

`float` can be used in place of the system floating-point type (usually `f64` or `f32`).

Using these make scripts more portable.
~~~

### Examples

```js
/// This 'do_update' can only be called on objects of type 'MyType' in method style
fn MyType.do_update(x, y) {
    this.update(x * y);
}

/// This 'do_update' can only be called on objects of type 'Strange-Type#Name::with_!@#symbols'
/// (which can be specified via 'Engine::register_type_with_name') in method style
fn "Strange-Type#Name::with_!@#symbols".do_update(x, y) {
    this.update(x * y);
}

/// Define a blanket version
fn do_update(x, y) {
    this = `${this}, ${x}, ${y}`;
}

/// This 'do_update' can only be called on integers in method style
fn int.do_update(x, y) {
    this += x * y
}

let obj = create_my_type();     // 'x' is 'MyType'

obj.type_of() == "MyType";

obj.do_update(42, 123);         // ok!

let x = 42;                     // 'x' is an integer

x.type_of() == "i64";

x.do_update(42, 123);           // ok!

let x = true;                   // 'x' is a boolean

x.type_of() == "bool";

x.do_update(42, 123);           // <- this works because there is a blanket version

// Use 'is_def_fn' with three parameters to test for typed methods
is_def_fn("MyType", "do_update", 2) == true;

is_def_fn("int", "do_update", 2) == true;
```


Bind to `this` for Module Functions
-----------------------------------

### The Problem

The _method-call_ syntax is not possible for [functions](functions.md) [imported](modules/import.md)
from [modules](modules/index.md).

```js
import "my_module" as foo;

let x = 42;

x.foo::change_value(1);     // <- syntax error
```

### The Solution

In order to call a [module](modules/index.md) [function](functions.md) as a method, it must be
defined with a restriction on the type of object pointed to by `this`:

```js
┌────────────────┐
│ my_module.rhai │
└────────────────┘

// This is a typed method function requiring 'this' to be an integer.
// Typed methods are automatically marked global when importing this module.
fn int.change_value(offset) {
    // 'this' is guaranteed to be an integer
    this += offset;
}


┌───────────┐
│ main.rhai │
└───────────┘

import "my_module";

let x = 42;

x.change_value(1);          // ok!

x == 43;
```
