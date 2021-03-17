Performance Build
=================

{{#include ../../links.md}}

Some features are for performance.  For example, using [`only_i32`] or [`only_i64`] disables all other integer types (such as `u16`).


Use Only One Integer Type
------------------------

If only a single integer type is needed in scripts &ndash; most of the time this is the case &ndash; it is best to avoid registering
lots of functions related to other integer types that will never be used.  As a result, [`Engine`] creation will be faster
because fewer functions need to be loaded.

The [`only_i32`] and [`only_i64`] features disable all integer types except `i32` or `i64` respectively.


Use Only 32-Bit Numbers
----------------------

If only 32-bit integers are needed &ndash; again, most of the time this is the case &ndash; turn on [`only_i32`].
Under this feature, only `i32` is supported as a built-in integer type and no others.

On 64-bit targets this may not gain much, but on certain 32-bit targets this improves performance
due to 64-bit arithmetic requiring more CPU cycles to complete.


Minimize Size of `Dynamic`
-------------------------

Turning on [`no_float`] or [`f32_float`] and [`only_i32`] on 32-bit targets makes the critical [`Dynamic`]
data type only 8 bytes long for 32-bit targets.

Normally [`Dynamic`] needs to be up 12-16 bytes long in order to hold an `i64` or `f64`.

A smaller [`Dynamic`] helps performance due to better cache efficiency.


Use `ImmutableString`
--------------------

Internally, Rhai uses _immutable_ [strings] instead of the Rust `String` type.  This is mainly to avoid excessive
cloning when passing function arguments.

Rhai's internal string type is `ImmutableString` (basically `Rc<String>` or `Arc<String>` depending on the [`sync`] feature).
It is cheap to clone, but expensive to modify (a new copy of the string must be made in order to change it).

Therefore, functions taking `String` parameters should use `ImmutableString` or `&str` (both map to `ImmutableString`)
for the best performance with Rhai.


Disable Closures
----------------

Support for [closures] that capture shared variables adds material overhead to script evaluation.

This is because every data access must be checked whether it is a shared value and, if so, take a read
lock before reading it.

As the vast majority of variables are _not_ shared, needless to say this is a non-trivial
performance overhead.

Use [`no_closure`] to disable closure and capturing support to optimize the hot path
because there is no need to take locks for shared data.
