Multi-Layer Functions
=====================

{{#include ../links.md}}


Usage Scenario
--------------

* A system is divided into separate _layers_, each providing logic in terms of scripted [functions].

* A lower layer provides _default implementations_ of certain functions.

* Higher layers each provide progressively more specific implementations of the same functions.

* A more specific function, if defined in a higher layer, always overrides the implementation in a lower layer.

* This is akin to object-oriented programming but with functions.

* This type of system is extremely convenient for dynamic business rules configuration, setting corporate-wide
  policies, granting permissions for specific roles etc. where specific, local rules need to override
  corporate-wide defaults.


Key Concepts
------------

* Each layer is a separate script.

* The lowest layer script is compiled into a base [`AST`].

* Higher layer scripts are also compiled into [`AST`] and _combined_ into the base using `AST::combine`
  (or the `+=` operator), overriding any existing functions.


Examples
--------

Assume the following four scripts:

```rust,no_run
+--------------+
| default.rhai |
+--------------+

// Default implementation of 'foo'.
fn foo(x) { x + 1 }

// Default implementation of 'bar'.
fn bar(x, y) { x + y }

// Default implementation of 'no_touch'.
fn no_touch() { throw "do not touch me!"; }


+-------------+
| lowest.rhai |
+-------------+

// Specific implementation of 'foo'.
fn foo(x) { x * 2 }

// New implementation for this layer.
fn baz() { print("hello!"); }


+-------------+
| middle.rhai |
+-------------+

// Specific implementation of 'bar'.
fn bar(x, y) { x - y }

// Specific implementation of 'baz'.
fn baz() { print("hey!"); }


+--------------+
| highest.rhai |
+--------------+

// Specific implementation of 'foo'.
fn foo(x) { x + 42 }
```

Load and combine them sequentially:

```rust,no_run
let engine = Engine::new();

// Compile the baseline default implementations.
let mut ast = engine.compile_file("default.rhai".into())?;

// Combine the first layer.
let lowest = engine.compile_file("lowest.rhai".into())?;
ast += lowest;

// Combine the second layer.
let middle = engine.compile_file("middle.rhai".into())?;
ast += lowest;

// Combine the third layer.
let highest = engine.compile_file("highest.rhai".into())?;
ast += lowest;

// Now, 'ast' contains the following functions:
//
// fn no_touch() {              // from 'default.rhai'
//     throw "do not touch me!";
// }
// fn foo(x) { x + 42 }         // from 'highest.rhai'
// fn bar(x, y) { x - y }       // from 'middle.rhai'
// fn baz() { print("hey!"); }  // from 'middle.rhai'
```

Unfortunately, there is no `super` call that calls the base implementation
(i.e. no way for a higher-layer function to call an equivalent lower-layer function).
