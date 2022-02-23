What Rhai Isn't
===============

{{#include ../links.md}}

Rhai's purpose is to provide a dynamic layer over Rust code, in the same spirit of _zero cost
abstractions_. It doesn't attempt to be a new language. For example:

* **No classes**.  Well, Rust doesn't either. On the other hand...

* **No traits**...  so it is also not Rust. Do your Rusty stuff in Rust.

* **No structures/records/tuples** &ndash; define your types in Rust instead; Rhai can seamlessly
  work with _any Rust type_ that implements `Clone`.

  There is, however, a built-in [object map] type which is adequate for most uses. It is possible to
  simulate [object-oriented programming (OOP)][OOP] by storing [function pointers] or [closures] in
  [object map] properties, turning them into _methods_.

* **No first-class functions** &ndash; Code your functions in Rust instead, and register them with Rhai.

  There is, however, support for simple [function pointers] to allow runtime dispatch by function name.

* **No garbage collection** &ndash; this should be expected, so...

* **No first-class closures** &ndash; do your closure magic in Rust instead:
  [turn a Rhai scripted function into a Rust closure]({{rootUrl}}/engine/call-fn.md).

  There is, however, support for simulated [closures] via [currying] a [function pointer] with
  [captured][closure] shared [variables].

* **No byte-codes/JIT** &ndash; Rhai has an optimized AST-walking interpreter which is fast enough
  for most casual usage scenarios. Essential AST data structures are packed and kept together to
  maximize cache friendliness.

  Functions are dispatched based on pre-calculated hashes and accessing [variables] are mostly
  through pre-calculated offsets to the [variables] file (a [`Scope`]), so it is seldom necessary to
  look something up by text name.
  
  In addition, Rhai's design deliberately avoids maintaining a _scope chain_ so [function] scopes do
  not pay any speed penalty.  This particular design also allows [variables] data to be kept
  together in a contiguous block, avoiding allocations and fragmentation while being cache-friendly.
  In a typical script evaluation run, no data is shared and nothing is locked.

  Still, the purpose of Rhai is not to be super _fast_, but to make it as easy and versatile as
  possible to integrate with native Rust applications.  What you lose from running an AST walker,
  you gain back from increased flexibility.

* **No formal language grammar** &ndash; Rhai uses a hand-coded lexer, a hand-coded top-down
  recursive-descent parser for statements, and a hand-coded Pratt parser for expressions.
  
  This lack of formalism allows the _tokenizer_ and _parser_ themselves to be exposed as services in
  order to support advanced features such as [disabling keywords and operators][disable keywords and operators],
  dynamically [changing tokens]({{rootUrl}}/engine/token-mapper.md) during parsing, adding
  [custom operators], defining [custom syntax] and [filtering variables definition]({{rootUrl}}/engine/def-var.md).

```admonish danger "Do not write the next 4D VR game in Rhai"

Due to this intended usage, Rhai deliberately keeps the language simple and small by omitting
advanced language features such as classes, inheritance, interfaces, generics, first-class
functions/closures, pattern matching, concurrency, byte-codes VM, JIT etc. Focus is on _flexibility_
and _ease of use_ instead of raw speed.

Avoid the temptation to write full-fledge application logic entirely in Rhai &ndash; that use case
is best fulfilled by more complete languages such as JavaScript or Lua.
```

```admonish tip "Tip: Use Rhai as a thin dynamic wrapper layer over Rust code"

In actual practice, it is usually best to expose a Rust API into Rhai for scripts to call.

All the core functionalities should be written in Rust, with Rhai being the dynamic _control_ layer.

This is similar to some dynamic languages where most of the core functionalities reside in a C/C++
standard library.

Another similar scenario is a web front-end driving back-end services written in a systems language.
In this case, JavaScript takes the role of Rhai while the back-end language, well... it can actually
also be Rust. Except that Rhai integrates with Rust _much_ more tightly, removing the need for
interfaces such as XHR calls and payload encoding such as JSON.
```
