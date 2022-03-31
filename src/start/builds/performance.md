Performance Build
=================

{{#include ../../links.md}}


Some features are for performance.  In order to squeeze out the maximum performance from Rhai, the
following features should be considered:

| Feature         | Description                                            | Rationale                 |
| --------------- | ------------------------------------------------------ | ------------------------- |
| [`only_i32`]    | support only a single `i32` integer type               | reduce data size          |
| [`no_float`]    | remove support for floating-point numbers              | reduce code size          |
| [`f32_float`]   | set floating-point numbers (if not disabled) to 32-bit | reduce data size          |
| [`no_closure`]  | remove support for variables sharing                   | no need for data locking  |
| [`unchecked`]   | disable all safety [checks][checked]                   | remove non-essential code |
| [`no_position`] | disable position tracking during parsing               | remove non-essential code |

When the above feature flags are used, performance may increase by around 15-20%.


Use Only One Integer Type
-------------------------

If only a single integer type is needed in scripts &ndash; most of the time this is the case &ndash;
it is best to avoid registering lots of functions related to other integer types that will never be used.
As a result, [`Engine`] creation will be faster because fewer functions need to be loaded.

The [`only_i32`] and [`only_i64`] features disable all integer types except `i32` or `i64` respectively.


Use Only 32-Bit Numbers
-----------------------

If only 32-bit integers are needed &ndash; again, most of the time this is the case &ndash; turn on [`only_i32`].
Under this feature, only `i32` is supported as a built-in integer type and no others.

On 64-bit targets this may not gain much, but on certain 32-bit targets this improves performance
due to 64-bit arithmetic requiring more CPU cycles to complete.


Minimize Size of `Dynamic`
--------------------------

Turning on [`f32_float`] (or [`no_float`]) and [`only_i32`] on 32-bit targets makes the critical
[`Dynamic`] data type only 8 bytes long for 32-bit targets.

Normally [`Dynamic`] needs to be up 12-16 bytes long in order to hold an `i64` or `f64`.

A smaller [`Dynamic`] helps performance due to better cache efficiency.


Use `ImmutableString`
---------------------

Internally, Rhai uses _immutable_ [strings] instead of the Rust `String` type.
This is mainly to avoid excessive cloning when passing function arguments.

Rhai's internal string type is [`ImmutableString`] (basically `Rc<SmartString>` or
`Arc<SmartString>` depending on the [`sync`] feature). It is cheap to clone, but expensive to modify
(a new copy of the string must be made in order to change it).

Therefore, functions taking `String` parameters should use [`ImmutableString`] or `&str`
(maps to [`ImmutableString`]) for the best performance with Rhai.


Disable Closures
----------------

Support for [closures] that capture _shared_ [variables] adds material overhead to script evaluation.

This is because every data access must be checked whether it is a shared value and, if so,
take a read lock before reading it.

As the vast majority of [variables] are _not_ shared, needless to say this is a non-trivial
performance overhead.

Use [`no_closure`] to disable support for [closures] to optimize the hot path because it no longer
needs to take locks for shared data.


Unchecked Build
---------------

By default, Rhai provides a [_Don't Panic_](https://en.wikipedia.org/wiki/Phrases_from_The_Hitchhiker%27s_Guide_to_the_Galaxy#Don't_Panic)
guarantee and prevents malicious scripts from bringing down the host. Any panic can be considered a bug.

For maximum performance, however, these [safety] checks can be turned off via the [`unchecked`] feature.


Disable Position
----------------

For embedded scripts that are not expected to cause errors, the [`no_position`] feature can be used
to disable position tracking during parsing.

No line number/character position information is kept for error reporting purposes.

This may result in a slightly fast build due to elimination of code related to position tracking.


Avoid Cloning
-------------

Rhai values are typically _cloned_ when passed around, especially into [function] calls.
Large data structures may incur material cloning overhead.

Some functions accept the first parameter as a mutable reference (i.e. `&mut`), for example
_methods_ for [custom types], and may avoid potentially-costly cloning.

For example, the `+=` (append) compound assignment takes a mutable reference to the [variable] while
the corresponding `+` (add) assignment usually doesn't.  The difference in performance can be huge:

```rust
let x = create_some_very_big_and_expensive_type();

x = x + 1;
//  ^ 'x' is cloned here

// The above is equivalent to:
let temp_value = x.clone() + 1;
x = temp_value;

x += 1;             // <- 'x' is NOT cloned
```

```admonish tip "Tip: Simple variable references are already optimized"

Rhai's script [optimizer][script optimization] is usually smart enough to _rewrite_ function calls
into [_method-call_]({{rootUrl}}/rust/methods.md) style or [_compound assignment_]({{rootUrl}}/language/assignment-op.md)
style to take advantage of this.

However, there are limits to its intelligence, and only **simple variable references** are optimized.

~~~rust
x = x + 1;          // <- this statement...

x += x;             // ... is rewritten as this

x[y] = x[y] + 1;    // <- but this is not, so this is MUCH slower...

x[y] += 1;          // ... than this

some_func(x, 1);    // <- this statement...

x.some_func(1);     // ... is rewritten as this

some_func(x[y], 1); // <- but this is not, so 'x[y]` is cloned
~~~
```


Short Variable Names for 32-Bit Systems
---------------------------------------

On 32-bit systems, [variable] and [constant] names longer than 11 ASCII characters incur additional
allocation overhead.

This is particularly true for local variables inside a hot loop, where they are created and destroyed
in rapid succession.

Therefore, avoid long [variable] and [constant] names that are over this limit.

On 64-bit systems, this limit is raised to 23 ASCII characters, which is almost always adequate.
