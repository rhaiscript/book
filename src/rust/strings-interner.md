Strings Interner
================

{{#include ../links.md}}


Because [strings] are immutable (i.e. the use the type [`ImmutableString`] instead of normal Rust `String`),
each operation on a [string] actually creates a new [`ImmutableString`] instance.

A _strings interner_ can substantially reduce memory usage by reusing the same [`ImmutableString`]
instance for the same [string] content.

An [`Engine`] contains a strings interner which is enabled by default
(disabled when using a [raw `Engine`]).

The maximum number of [strings] to be interned can be set via
[`Engine::set_max_strings_interned`][options] (set to zero to disable the strings interner).
