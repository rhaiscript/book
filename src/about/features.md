Features of Rhai
================

{{#include ../links.md}}


```admonish success "Easy"

* Simple language similar to JavaScript+Rust with dynamic typing.

* Tight integration with native Rust [functions] and [types][custom types] including
  [getters/setters], [methods] and [indexers].

* Freely pass Rust values into a script as [variables]/[constants] via an external [`Scope`] &ndash;
  all clonable Rust types are supported seamlessly without the need to implement any special trait.

* Easily [call a script-defined function]({{rootUrl}}/engine/call-fn.md) from Rust.

* Very few additional dependencies &ndash; right now only [`smallvec`](https://crates.io/crates/smallvec),
  [`thin-vec`](https://crates.io/crates/thin-vec), [`num-traits`](https://crates.io/crates/num-traits),
  [`ahash`](https://crates.io/crates/ahash), [`bitflags`](https://crates.io/crates/bitflags) and [`smartstring`];
  for [`no-std`] and [WASM] builds, a number of additional dependencies are pulled in to provide for missing functionalities.

* [Plugins] system powered by procedural macros simplifies custom API development.
```

```admonish danger "Fast"

* Fairly efficient evaluation &ndash; 1 million iterations in 0.14 sec on a single-core, 2.6 GHz Linux VM
  running [this script](https://github.com/rhaiscript/rhai/blob/main/scripts/speed_test.rhai)
  (also see [benchmarks](benchmarks.md)).

* Compile once to [AST][`AST`] for repeated evaluations.

* Scripts are [optimized][script optimization] &ndash; useful for template-based machine-generated scripts.
```

```admonish tip "Dynamic"

* [Function overloading]({{rootUrl}}/language/overload.md).

* [Operator overloading]({{rootUrl}}/rust/operators.md).

* Organize code base with dynamically-loadable [modules], optionally overriding the
  [resolution][module resolver] process.

* Dynamic dispatch via [function pointers] with additional support for [currying].

* [Closures] that can capture shared variables.

* Some support for [object-oriented programming (OOP)][OOP].

* Hook into variables access via a [variable resolver], or control definition of variables via a
  [variable definition filter].
```

```admonish warning "Safe"

* Relatively little `unsafe` code &ndash; yes there are some for performance reasons.

* Sand-boxed &ndash; the scripting [`Engine`], if declared immutable, cannot mutate the containing
  environment unless [explicitly permitted]({{rootUrl}}/patterns/control.md).

* Passes Miri.
```

```admonish bug "Rugged"

* [_Don't Panic_][safety] guarantee &ndash; Any panic is a bug. It never panics the host system.

* Protected against malicious attacks &ndash; such as [stack-overflow][maximum call stack depth],
  [over-sized data][maximum length of strings], and [runaway scripts][maximum number of operations]
  etc. &ndash; that may come from untrusted third-party user-land scripts.

* Track script evaluation [progress] and manually terminate a script run.
```

```admonish example "Flexible"

* Re-entrant scripting [`Engine`] can be made `Send + Sync` (via the [`sync`] feature).

* Support for [`Decimal`][rust_decimal] numbers.

* Serialization/deserialization support via [`serde`](https://crates.io/crates/serde).

* Support for [minimal builds] by excluding unneeded language [features].

* Supports [most build targets](targets.md) including `no-std` and [WASM].

* Surgically [disable keywords and operators] to restrict the language.

* Use as a [DSL] by defining [custom operators] and/or extending the language with [custom syntax].

* A [debugging][debugger] interface provides powerful debugging support.
```
