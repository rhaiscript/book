What Rhai Isn't
===============

{{#include ../links.md}}

Rhai's purpose is to provide a dynamic layer over Rust code, in the same spirit of _zero cost abstractions_.

It doesn't attempt to be a new language. For example:

* **No classes**.  Well, Rust doesn't either. On the other hand...

* **No traits**...  so it is also not Rust. Do your Rusty stuff in Rust.

* **No structures/records/tuples** &ndash; define your types in Rust instead; Rhai can seamlessly
  work with [_any Rust type_][custom types] that implements `Clone`.

```admonish tip.small "Object maps"
There is a built-in [object map] type which is adequate for most uses.

It is also possible to simulate [object-oriented programming (OOP)][OOP] by storing [function pointers]
or [closures] in [object map] properties, turning them into _[methods]_.
```

* **No first-class functions** &ndash; Code your functions in Rust instead, and register them with Rhai.

```admonish tip.small "Function pointers"

There is support for simple [function pointers] to allow runtime dispatch by function name.
```

* **No garbage collection** &ndash; this should be expected, so...

* **No first-class closures** &ndash; do your closure magic in Rust instead:
  [turn a Rhai scripted function into a Rust closure]({{rootUrl}}/engine/call-fn.md).

```admonish tip.small "Simulated closures"

There is support for simulated [closures] via [currying] a [function pointer] with
[captured][closure] shared [variables].
```

* **No formal language grammar** &ndash; Rhai uses a hand-coded lexer, a hand-coded top-down
  recursive-descent parser for statements, and a hand-coded Pratt parser for expressions.

```admonish tip.small "Highly customizable"
This lack of formalism allows the _tokenizer_ and _parser_ themselves to be exposed as services in
order to support a wide range of user customizations, such as:

* [disabling keywords and operators][disable keywords and operators],
* dynamically [changing tokens]({{rootUrl}}/engine/token-mapper.md) during parsing,
* adding [custom operators],
* defining [custom syntax],
* [filtering variables definition][variable definition filter].
```

* **No bytecodes/JIT** &ndash; Rhai uses a heavily-optimized AST-walking interpreter which is fast
  enough for most real-life scenarios.

```admonish info.small "How it compares?"

See Rhai performance [benchmarks].
```
