Global Constants
================

{{#include ../links.md}}


```admonish info "Usage scenario"

* Script has a lot of duplicated [constants] used inside [functions].

* For easier management, [constants] are declared at the top of the script.

* As Rhai [functions] are pure, they cannot access [constants] declared at global level
  except through [`global`].

* Sprinkling large number of [`global::CONSTANT`][`global`] throughout the script makes
  it slow and cumbersome.

* Using [`global`] or a [variable resolver] defeats
  [constants propagation]({{rootUrl}}/engine/optimize/constants.md) in [script optimization].
```

```admonish abstract "Key concepts"

* The key to global [constants] is to use them to [optimize][script optimization] a script.
  Otherwise, it would be just as simple to pass the constants into a custom [`Scope`] instead.

* The script is first compiled into an [`AST`], and all [constants] are extracted.

* The [constants] are then supplied to [re-optimize][script optimization] the [`AST`].

* This pattern also works under [_Strict Variables Mode_][strict variables].
```


Example
-------

Assume that the following Rhai script needs to work (but it doesn't).

```rust
// These are constants

const FOO = 1;
const BAR = 123;
const MAGIC_NUMBER = 42;

fn get_magic() {
    MAGIC_NUMBER        // <- oops! 'MAGIC_NUMBER' not found!
}

fn calc_foo(x) {
    x * global::FOO     // <- works but cumbersome; not desirable!
}

let magic = get_magic() * BAR;

let x = calc_foo(magic);

print(x);
```


Step 1 &ndash; Compile Script into `AST`
----------------------------------------

Compile the script into [`AST`] form.

Normally, it is useful to disable [optimizations][script optimization] at this stage since
the [`AST`] will be re-optimized later.

[_Strict Variables Mode_][strict variables] must be OFF for this to work.

```rust
// Turn Strict Variables Mode OFF (if necessary)
engine.set_strict_variables(false);

// Turn optimizations OFF
engine.set_optimization_level(OptimizationLevel::None);

let ast = engine.compile("...")?;
```


Step 2 &ndash; Extract Constants
--------------------------------

Use [`AST::iter_literal_variables`](https://docs.rs/rhai/{{version}}/rhai/struct.AST.html#method.iter_literal_variables)
to extract top-level [constants] from the [`AST`].

```rust
let mut scope = Scope::new();

// Extract all top-level constants without running the script
ast.iter_literal_variables(true, false).for_each(|(name, _, value)|
    scope.push_constant(name, value);
);

// 'scope' now contains: FOO, BAR, MAGIC_NUMBER
```


Step 3a &ndash; Propagate Constants
-----------------------------------

[Re-optimize][script optimization] the [`AST`] using the new constants.

```rust
// Turn optimization back ON
engine.set_optimization_level(OptimizationLevel::Simple);

let ast = engine.optimize_ast(&scope, ast, engine.optimization_level());
```


Step 3b &ndash; Recompile Script (Alternative)
----------------------------------------------

If [_Strict Variables Mode_][strict variables] is used, however, it is necessary to re-compile the
script in order to detect undefined [variable] usages.

```rust
// Turn Strict Variables Mode back ON
engine.set_strict_variables(true);

// Turn optimization back ON
engine.set_optimization_level(OptimizationLevel::Simple);

// Re-compile the script using constants in 'scope'
let ast = engine.compile_with_scope(&scope, "...")?;
```


Step 4 &ndash; Run the Script
-----------------------------

At this step, the [`AST`] is now optimized with constants propagated into all access sites.

The script essentially becomes:

```rust
// These are constants

const FOO = 1;
const BAR = 123;
const MAGIC_NUMBER = 42;

fn get_magic() {
    42      // <- constant replaced by value
}

fn calc_foo(x) {
    x * global::FOO
}

let magic = get_magic() * 123;  // <- constant replaced by value

let x = calc_foo(magic);

print(x);
```

Run it via `Engine::run_ast` or `Engine::eval_ast`.

```rust
// The 'scope' is no longer necessary
engine.run_ast(&ast)?;
```
