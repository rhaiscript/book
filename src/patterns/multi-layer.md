Multi-Layered Functions
=======================

{{#include ../links.md}}


```admonish info "Usage scenario"

* A system is divided into separate _layers_, each providing logic in terms of scripted [functions].

* A lower layer provides _default implementations_ of certain [functions].

* Higher layers each provide progressively more specific implementations of the same [functions].

* A more specific [function], if defined in a higher layer, always overrides the implementation
  in a lower layer.

* This is akin to object-oriented programming but with [functions].

* This type of system is extremely convenient for dynamic business rules configuration, setting
  corporate-wide policies, granting permissions for specific roles etc. where specific, local rules
  need to override corporate-wide defaults.
```

```admonish tip "Practical scenario"

Assuming a LOB (line-of-business) system for a large MNC (multi-national corporation) with branches,
facilities and offices across the globe.

The system needs to provide basic, corporate-wide policies to be enforced through the worldwide
organization, but also cater for country- or region-specific rules, practices and regulations.

|    Layer    | Description                                         |
| :---------: | --------------------------------------------------- |
| `corporate` | corporate-wide policies                             |
| `regional`  | regional policy overrides                           |
|  `country`  | country-specific modifications for legal compliance |
|  `office`   | special treatments for individual office locations  |
```

```admonish abstract "Key concepts"

* Each layer is a separate script.

* The lowest layer script is compiled into a base [`AST`].

* Higher layer scripts are also compiled into [`AST`] and _combined_ into the base using
  [`AST::combine`]({{rootUrl}}/engine/ast.md) (or the `+=` operator), overriding any existing [functions].
```


Examples
--------

Assume the following four scripts, one for each layer:

```rust,no_run
┌────────────────┐
│ corporate.rhai │
└────────────────┘

// Default implementation of 'foo'.
fn foo(x) { x + 1 }

// Default implementation of 'bar'.
fn bar(x, y) { x + y }

// Default implementation of 'no_touch'.
fn no_touch() { throw "do not touch me!"; }


┌───────────────┐
│ regional.rhai │
└───────────────┘

// Specific implementation of 'foo'.
fn foo(x) { x * 2 }

// New implementation for this layer.
fn baz() { print("hello!"); }


┌──────────────┐
│ country.rhai │
└──────────────┘

// Specific implementation of 'bar'.
fn bar(x, y) { x - y }

// Specific implementation of 'baz'.
fn baz() { print("hey!"); }


┌─────────────┐
│ office.rhai │
└─────────────┘

// Specific implementation of 'foo'.
fn foo(x) { x + 42 }
```

Load and combine them sequentially:

```rust,no_run
let engine = Engine::new();

// Compile the baseline layer.
let mut ast = engine.compile_file("corporate.rhai".into())?;

// Combine the first layer.
let lowest = engine.compile_file("regional.rhai".into())?;
ast += lowest;

// Combine the second layer.
let middle = engine.compile_file("country.rhai".into())?;
ast += lowest;

// Combine the third layer.
let highest = engine.compile_file("office.rhai".into())?;
ast += lowest;

// Now, 'ast' contains the following functions:
//
// fn no_touch() {                // from 'corporate.rhai'
//     throw "do not touch me!";
// }
// fn foo(x) { x + 42 }           // from 'regional.rhai'
// fn bar(x, y) { x - y }         // from 'country.rhai'
// fn baz() { print("hey!"); }    // from 'office.rhai'
```

```admonish failure.small "No super call"

Unfortunately, there is no `super` call that calls the base implementation (i.e. no way for a
higher-layer function to call an equivalent lower-layer function).
```
