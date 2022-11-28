Benchmarking Rhai
=================

{{#include ../links.md}}

[bytecodes]: https://en.wikipedia.org/wiki/Bytecode
[JIT]: https://en.wikipedia.org/wiki/Just-in-time_compilation
[Python 3]: https://www.python.org
[V8]: https://v8.dev
[Node.js]: https://nodejs.org
[Lua]: https://www.lua.org/
[LuaJIT]: https://luajit.org

```admonish tip.side.wide "Tip: Nothing beats a JIT"

Needless to say [V8] (JavaScript), which is a [JIT] compiler with type specialization, blows any interpreter
(AST or [bytecodes] based) out of the water, as also should [LuaJIT].

So if you _absolutely must_ have top performance...
```

The purpose of Rhai is not to be _blazing fast_, but to make it as easy and versatile as possible to
integrate with native Rust applications.
  
What you lose from running an [AST][`AST`] walker, you gain back from increased flexibility.

The following benchmarks were run on a 2.6GHz Linux VM comparing
[performance-optimized](../start/builds/performance.md) and full builds of Rhai with [Python 3] and
[V8] ([Node.js]).

| Benchmark                                         | Rhai<br/>(Perf) | Rhai</br>(Full) | [Python 3]<br/>([bytecodes]) | [V8]<br/>([JIT]) | Description                                                                     |
| ------------------------------------------------- | :-------------: | :-------------: | :--------------------------: | :--------------: | ------------------------------------------------------------------------------- |
| [Fibonacci]({{repoHome}}/scripts/fibonacci.rhai)  |      2.4s       |      3.4s       |             0.6s             |      0.07s       | stresses recursive [function] calls                                             |
| [1M loop]({{repoHome}}/scripts/speed_test.rhai)   |      0.14s      |      0.26s      |            0.08s             |      0.05s       | a simple counting loop (1 million iterations) that must run as fast as possible |
| [Prime numbers]({{repoHome}}/scripts/primes.rhai) |       1s        |      1.5s       |             0.4s             |      0.09s       | a closer-to-real-life calculation workload                                      |

In general, Rhai is roughly 2.5x slower than [Python 3], which is a [bytecodes] interpreter, for
typical real-life workloads.


```admonish question "TL;DR &ndash; Rhai is usually fast enough"

#### Small data structures

Essential [AST][`AST`] and runtime data structures are packed small and kept together
to maximize cache friendliness.

#### Pre-calculations

[Functions] are dispatched based on pre-calculated hashes.
[Variables] are mostly accessed through pre-calculated offsets to the [variables] file (a [`Scope`]).
It is seldom necessary to look something up by name.

#### Caching

[Function] resolutions are cached so they do not incur lookup costs after a couple of calls.

#### No scope-chain

Maintaining a _scope chain_ is deliberately avoided by design so [function] scopes do
not pay any speed penalty.
This allows [variables] data to be kept together in a contiguous block, avoiding allocations
and fragmentation while being cache-friendly.

#### Immutable strings

Rhai uses [immutable strings][`ImmutableString`] to bypass cloning issues.

#### No sharing

In a typical script evaluation run, no data is shared and nothing is locked (other than
[variables] captured by [closures]).
```

```admonish danger "DO NOT: Write the next 4D VR game entirely in Rhai"

Rhai deliberately keeps the language small and lean by omitting advanced language features
such as classes, inheritance, interfaces, generics, first-class functions/closures, pattern matching,
monads (whatever), concurrency, async etc.

Focus is on _flexibility_ and _ease of use_ instead of a powerful, expressive language.

Avoid the temptation to write full-fledge application logic entirely in Rhai &ndash; that use case
is best fulfilled by more complete scripting languages such as JavaScript or [Lua].
```

```admonish tip "DO THIS: Use Rhai as a thin dynamic wrapper layer over Rust code"

In actual practice, it is usually best to expose a Rust API into Rhai for scripts to call.

All the core functionalities should be written in Rust, with Rhai being the dynamic _control_ layer.

This is similar to some dynamic languages where most of the core functionalities reside in a C/C++
standard library (e.g. [Python 3]).

Another similar scenario is a web front-end driving back-end services written in a systems language.
In this case, JavaScript takes the role of Rhai while the back-end language, well... it can actually
also be Rust. Except that Rhai integrates with Rust _much_ more tightly, removing the need for
interfaces such as XHR calls and payload encoding such as JSON.
```
