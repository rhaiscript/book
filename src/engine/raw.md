Raw `Engine`
===========

{{#include ../links.md}}

`Engine::new` creates a scripting [`Engine`] with common functionalities (e.g. printing to the console via `print`).

In many controlled embedded environments, however, these may not be needed and unnecessarily occupy
application code storage space.

Use `Engine::new_raw` to create a _raw_ `Engine`, in which only a minimal set of
basic arithmetic and logical operators are supported (see below).

To add more functionalities to a _raw_ `Engine`, load [packages] into it.


Built-in Operators
------------------

Even with a raw [`Engine`], some operators are built in and are always available.

See [_Built-in Operators_][built-in operators] for a full list.
