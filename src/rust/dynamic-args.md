`Dynamic` Parameters in Rust Functions
=====================================

{{#include ../links.md}}

It is possible for Rust functions to contain parameters of type [`Dynamic`].
Any clonable value can be set into a [`Dynamic`] value.

Any parameter in a registered Rust function with a specific type has higher precedence over the
[`Dynamic`] type, so it is important to understand which _version_ of a function will be used.

For example, the `push` method of an [array] is implemented as follows (minus code that protects
against [over-sized arrays][maximum size of arrays]), making the function applicable for all
item types.

```rust,no_run
// 'item: Dynamic' matches all data types
fn push(array: &mut Array, item: Dynamic) {
    array.push(item);
}
```


Examples
--------

A [`Dynamic`] value has less precedence than a value of a specific type, and parameter matching starts
from the left to the right. Candidate functions will be matched in order of parameter types.

Therefore, always leave [`Dynamic`] parameters as far to the right as possible.

```rust,no_run
use rhai::{Engine, Dynamic};

// Different versions of the same function 'foo'
// will be matched in the following order.

fn foo1(x: i64, y: &str, z: bool) { }

fn foo2(x: i64, y: &str, z: Dynamic) { }

fn foo3(x: i64, y: Dynamic, z: bool) { }

fn foo4(x: i64, y: Dynamic, z: Dynamic) { }

fn foo5(x: Dynamic, y: &str, z: bool) { }

fn foo6(x: Dynamic, y: &str, z: Dynamic) { }

fn foo7(x: Dynamic, y: Dynamic, z: bool) { }

fn foo8(x: Dynamic, y: Dynamic, z: Dynamic) { }

let mut engine = Engine::new();

// Register all functions under the same name (order does not matter)

engine.register_fn("foo", foo5)
      .register_fn("foo", foo7)
      .register_fn("foo", foo2)
      .register_fn("foo", foo8)
      .register_fn("foo", foo1)
      .register_fn("foo", foo3)
      .register_fn("foo", foo6)
      .register_fn("foo", foo4);
```


~~~admonish warning "Only the right-most 16 parameters can be `Dynamic`"

The number of parameter permutations go up exponentially, and therefore there is a realistic limit
of 16 parameters allowed to be [`Dynamic`], counting from the _right-most side_.

For example, Rhai will not find the following function &ndash; Oh! and those 16 parameters to the right
certainly have nothing to do with it!

```rust,no_run
// The 'd' parameter counts 17th from the right!
fn weird(a: i64, d: Dynamic, x1: i64, x2: i64, x3: i64, x4: i64,
                             x5: i64, x6: i64, x7: i64, x8: i64,
                             x9: i64, x10: i64, x11: i64, x12: i64,
                             x13: i64, x14: i64, x15: i64, x16: i64) {

    // ... do something unspeakably evil with all those parameters ...
}
```
~~~


TL;DR
-----

```admonish question "How is this implemented?"

#### Hash lookup

Since functions in Rhai can be [overloaded][function overloading], Rhai uses a single _hash_ number
to quickly lookup the actual function, based on argument types.

For each function call, a hash is calculated made up from:

1) the function's [namespace], if any,
2) the function's name,
3) number of arguments,
4) the unique ID of the type of each argument, if any.

The correct function is then obtained via a simple hash lookup.

#### Limitations

This method is _fast_, but at the expense of flexibility (such as multiple argument types that must
map to a single version).  That is because each type has a different ID, and thus they calculate to
different hash numbers.

This is the reason why [generic functions](generic.md) must be expanded into concrete types.

The type ID of [`Dynamic`] is different from any other type, but it must match all types seamlessly.
Needless to say, this creates a slight problem.

#### Trying combinations

If the combined hash calculated from the actual argument type ID's is not found, then the [`Engine`]
calculates hashes for different _combinations_ of argument types and [`Dynamic`], systematically
replacing different arguments with [`Dynamic`] _starting from the right-most parameter_.

Thus, assuming a three-argument function call:

~~~rust,no_run
foo(42, "hello", true);
~~~

The following hashes will be calculated, in order.
They will be _all different_.

| Order | Hash calculation method                             |
| :---: | --------------------------------------------------- |
|   1   | `foo` + 3 + `i64` + `string` + `bool`               |
|   2   | `foo` + 3 + `i64` + `string` + [`Dynamic`]          |
|   3   | `foo` + 3 + `i64` + [`Dynamic`] + `bool`            |
|   4   | `foo` + 3 + `i64` + [`Dynamic`] + [`Dynamic`]       |
|   5   | `foo` + 3 + [`Dynamic`] + `string` + `bool`         |
|   6   | `foo` + 3 + [`Dynamic`] + `string` + [`Dynamic`]    |
|   7   | `foo` + 3 + [`Dynamic`] + [`Dynamic`] + `bool`      |
|   8   | `foo` + 3 + [`Dynamic`] + [`Dynamic`] + [`Dynamic`] |

Therefore, the version with all the correct parameter types will always be found first if it exists.

At soon as a hash is found, the process stops.

Otherwise, it goes on for up to 16 arguments, or at most 256 tries.
That's where the 16 parameters limit comes from.
```

```admonish question "What?! It calculates 256 hashes for each function call???!!!"

Of course not.

Function hashes are _cached_, so this process only happens _once_, and only up to the number of
rounds for the correct function to be found.

If not, then yes, it will calculate up to 2<sup>_n_</sup> hashes where _n_ is the number of
arguments (up to 16). But again, this will only be done _once_ for that particular
combination of argument types.
```
