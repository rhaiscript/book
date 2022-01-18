The Rhai Scripting Language
==========================

[What is Rhai](about/index.md)

----------------------

User Guide
==========

- [Features of Rhai](about/features.md)
  - [Supported Targets and Builds](about/targets.md)
  - [What Rhai Isn't](about/non-design.md)
  - [Licensing](about/license.md)
  - [Related Resources](about/related.md)
- [Getting Started](start/index.md)
  - [Online Playground](start/playground.md)
  - [Install the Rhai Crate](start/install.md)
  - [Optional Features](start/features.md)
  - [Packaged Utilities](start/bin.md)
- [Using the `Engine`](engine/index.md)
  - ["Hello, Rhai"](engine/hello-world.md)
  - [Compile to AST](engine/compile.md)
  - [Raw Engine](engine/raw.md)
    - [Built-in Operators](engine/builtin.md)
  - [Scope &ndash; Maintaining State](engine/scope.md)
  - [Expressions Only](engine/expressions.md)
  - [Options](engine/options.md)
- [Examples](start/examples/index.md)
  - [Rust](start/examples/rust.md)
  - [Scripts](start/examples/scripts.md)
- [Special Builds](start/builds/index.md)
  - [Performance](start/builds/performance.md)
  - [Minimal](start/builds/minimal.md)
  - [no-std](start/builds/no-std.md)
  - [WebAssembly (WASM)](start/builds/wasm.md)

----------------------

Rust Integration
================

- [Introduction](rust/index.md)
- [Traits](rust/traits.md)
- [Register a Rust Function](rust/functions.md)
  - [Dynamic Parameters](rust/dynamic-args.md)
  - [String Parameters](rust/strings.md)
  - [Generic Functions](rust/generic.md)
  - [Fallible Functions](rust/fallible.md)
  - [`NativeCallContext`](rust/context.md)
- [Call a Rhai Function from Rust](engine/call-fn.md)
  - [Create a Rust Closure from a Rhai Function](engine/func.md)
- [Override a Built-in Function](rust/override.md)
- [Operator Overloading](rust/operators.md)
- [Register any Rust Type and its Methods](rust/custom.md)
  - [Property Getters and Setters](rust/getters-setters.md)
  - [Indexers](rust/indexers.md)
  - [Call Method as Function](rust/methods.md)
  - [Collection Types](rust/collections.md)
  - [Disable Custom Types](rust/disable-custom.md)
  - [Printing Custom Types](rust/print-custom.md)
- [Modules](rust/modules/index.md)
  - [Create in Rust](rust/modules/create.md)
  - [Create from AST](rust/modules/ast.md)
  - [Use a Module](rust/modules/use.md)
  - [Module Resolvers](rust/modules/resolvers.md)
    - [Custom Module Resolvers](rust/modules/imp-resolver.md)
  - [Self-Contained AST](rust/modules/self-contained.md)
- [Plugins](plugins/index.md)
  - [Export a Rust Module](plugins/module.md)
  - [Export a Rust Function](plugins/function.md)
- [Packages](rust/packages/index.md)
  - [Built-in Packages](rust/packages/builtin.md)
  - [Create Custom Packages](rust/packages/create.md)
  - [Create Packages as Crates](rust/packages/crate.md)
  - [External Packages](lib/index.md)
    - [Random Number Generation, Shuffling and Sampling](lib/rhai-rand.md)

----------------------

Language Reference
==================

- [Comments](language/comments.md)
  - [Doc-Comments](language/doc-comments.md)
- [Values and Types](language/values-and-types.md)
  - [Dynamic Values](language/dynamic.md)
    - [type_of()](language/type-of.md)
    - [Value Tag](language/dynamic-tag.md)
    - [Serialization/Deserialization with `serde`](rust/serde.md)
  - [Numbers](language/numbers.md)
    - [Operators](language/num-op.md)
    - [Functions](language/num-fn.md)
    - [Value Conversions](language/convert.md)
    - [Ranges](language/ranges.md)
    - [Bit-Fields](language/bit-fields.md)
  - [Strings and Characters](language/strings-chars.md)
    - [`ImmutableString`](rust/immutable-string.md)
    - [Standard Functions](language/string-fn.md)
  - [Arrays](language/arrays.md)
    - [BLOB's (Byte Arrays)](language/blobs.md)
  - [Object Maps](language/object-maps.md)
    - [Parse from JSON](language/json.md)
    - [Special Support for OOP](language/object-maps-oop.md)
  - [Timestamps](language/timestamps.md)
- [Keywords](language/keywords.md)
- [Statements](language/statements.md)
- [Assignments](language/assignment.md)
- [Variables](language/variables.md)
  - [Strict Variables Mode](engine/strict-var.md)
  - [Variable Resolver](engine/var.md)
- [Constants](language/constants.md)
  - [Automatic Global Module](language/global.md)
- [Logic Operators](language/logic.md)
- [Compound Assignments](language/assignment-op.md)
- [If Statement](language/if.md)
  - [If Expression](language/if-expression.md)
- [In Operator](language/in.md)
- [Switch Expression](language/switch.md)
- [While Loop](language/while.md)
- [Do Loop](language/do.md)
- [Loop Statement](language/loop.md)
- [For Loop](language/for.md)
  - [Make a Custom Type Iterable](language/iterator.md)
- [Return Value](language/return.md)
- [Throw Exception on Error](language/throw.md)
  - [Catch Exception](language/try-catch.md)
- [Functions](language/functions.md)
  - [Method Calls](language/fn-method.md)
  - [Overloading](language/overload.md)
  - [Namespaces](language/fn-namespaces.md)
  - [Call Within Caller's Scope](language/fn-parent-scope.md)
  - [Function Pointers](language/fn-ptr.md)
    - [Currying](language/fn-curry.md)
    - [Anonymous Functions](language/fn-anon.md)
    - [Closures](language/fn-closure.md)
  - [Metadata](engine/metadata/index.md)
    - [Get Functions Metadata in Rhai](language/fn-metadata.md)
    - [Get Function Signatures in Rust](engine/metadata/gen_fn_sig.md)
    - [Export Functions Metadata to JSON](engine/metadata/export_to_json.md)
- [Print and Debug](language/print-debug.md)
- [Modules](language/modules/index.md)
  - [Export Variables, Functions and Sub-Modules](language/modules/export.md)
  - [Import Modules](language/modules/import.md)
- [Eval Function](language/eval.md)

----------------------

Safety and Protection
=====================

- [Introduction](safety/index.md)
- [Sand-Boxing](safety/sandbox.md)
- [Maximum Length of Strings](safety/max-string-size.md)
- [Maximum Size of Arrays](safety/max-array-size.md)
- [Maximum Size of Object Maps](safety/max-map-size.md)
- [Maximum Number of Operations](safety/max-operations.md)
  - [Tracking Progress and Force-Termination](safety/progress.md)
- [Maximum Number of Modules](safety/max-modules.md)
- [Maximum Call Stack Depth](safety/max-call-stack.md)
- [Maximum Statement Depth](safety/max-stmt-depth.md)
- [Turn Off Safety Checks](safety/checked.md)

----------------------

Script Optimization
===================

- [Introduction](engine/optimize/index.md)
- [Optimization Levels](engine/optimize/optimize-levels.md)
- [Re-Optimize an AST](engine/optimize/reoptimize.md)
- [Eager Function Evaluation](engine/optimize/eager.md)
- [Side-Effect Considerations](engine/optimize/side-effects.md)
- [Volatility Considerations](engine/optimize/volatility.md)
- [Subtle Semantic Changes](engine/optimize/semantics.md)

----------------------

Advanced Topics
===============

- [Manage AST's](engine/ast.md)
- [Low-Level API to Register Functions](rust/register-raw.md)
- [Use Rhai as a DSL](engine/dsl.md)
  - [Remap Tokens During Parsing](engine/token-mapper.md)
  - [Disable Keywords and/or Operators](engine/disable-keywords.md)
  - [Disable Looping](engine/disable-looping.md)
  - [Custom Operators](engine/custom-op.md)
  - [Extend with Custom Syntax](engine/custom-syntax.md)

----------------------

Usage Patterns
==============

- [Object-Oriented Programming (OOP)](patterns/oop.md)
- [Working With Rust Enums](patterns/enums.md)
- [Loadable Configuration](patterns/config.md)
- [Hot Reloading](patterns/hot-reload.md)
- [Control Layer Over Rust Backend](patterns/control.md)
- [Singleton Command](patterns/singleton.md)
- [Multi-Layer Functions](patterns/multi-layer.md)
- [One Engine Instance Per Call](patterns/parallel.md)
- [Multi-Threaded Synchronization](patterns/multi-threading.md)
- [Scriptable Event Handler with State](patterns/events.md)
  - [Main Style](patterns/events-1.md)
  - [JS Style](patterns/events-2.md)
  - [Map Style](patterns/events-3.md)
- [Dynamic Constants Provider](patterns/dynamic-const.md)
- [Multiple Instantiation](patterns/multiple.md)

----------------------

Appendix
========

- [External Tools](tools/index.md)
  - [Online Playground](tools/playground.md)
  - [`rhai-doc`](tools/rhai-doc.md)
- [Keywords](appendix/keywords.md)
- [Operators and Symbols](appendix/operators.md)
- [Literals](appendix/literals.md)
