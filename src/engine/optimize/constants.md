Constants Propagation
=====================

{{#include ../../links.md}}

```admonish tip.side.wide "Usage"

Effective in template-based machine-generated scripts to turn on/off certain sections.
```

[Constants] propagation is commonly used to:

* remove dead code,

* avoid [variable] lookups,

* [pre-calculate](op-eval.md) [constant] expressions.

```rust,no_run
const ABC = true;
const X = 41;

if ABC || calc(X+1) { print("done!"); }     // 'ABC' is constant so replaced by 'true'...
                                            // 'X' is constant so replaced by 41... 

if true || calc(42) { print("done!"); }     // '41+1' is replaced by 42
                                            // since '||' short-circuits, 'calc' is never called

if true { print("done!"); }                 // <- the line above is equivalent to this

print("done!");                             // <- the line above is further simplified to this
                                            //    because the condition is always true
```

~~~admonish tip "Tip: Custom `Scope` constants"

[Constant] values can be provided in a custom [`Scope`] object to the [`Engine`]
for optimization purposes.

```rust,no_run
use rhai::{Engine, Scope};

let engine = Engine::new();

let mut scope = Scope::new();

// Add constant to custom scope
scope.push_constant("ABC", true);

// Evaluate script with custom scope
engine.run_with_scope(&mut scope,
r#"
    if ABC {    // 'ABC' is replaced by 'true'
        print("done!");
    }
"#)?;
```
~~~

~~~admonish tip "Tip: Customer module constants"

[Constants] defined in [modules] that are registered into an [`Engine`] via
`Engine::register_global_module` are used in optimization.

```rust,no_run
use rhai::{Engine, Module};

let mut engine = Engine::new();

let mut module = Module::new();

// Add constant to module
module.set_var("ABC", true);

// Register global module
engine.register_global_module(module.into());

// Evaluate script
engine.run(
r#"
    if ABC {    // 'ABC' is replaced by 'true'
        print("done!");
    }
"#)?;
```
~~~

~~~admonish danger "Caveat: Constants in custom scope and modules are also propagated into functions"

[Constants] defined at _global_ level typically cannot be seen by script [functions] because they are _pure_.

```rust,no_run
const MY_CONSTANT = 42;     // <- constant defined at global level

print(MY_CONSTANT);         // <- optimized to: print(42)

fn foo() {
    MY_CONSTANT             // <- not optimized: 'foo' cannot see 'MY_CONSTANT'
}

print(foo());               // error: 'MY_CONSTANT' not found
```

When [constants] are provided in a custom [`Scope`] (e.g. via `Engine::compile_with_scope`,
`Engine::eval_with_scope` or `Engine::run_with_scope`), or in a [module] registered via
`Engine::register_global_module`, instead of defined within the same script, they are also
propagated to [functions].

This is usually the intuitive usage and behavior expected by regular users, even though it means
that a script will behave differently (essentially a runtime error) when [script optimization] is disabled.

```rust,no_run
use rhai::{Engine, Scope};

let engine = Engine::new();

let mut scope = Scope::new();

// Add constant to custom scope
scope.push_constant("MY_CONSTANT", 42_i64);

engine.run_with_scope(&mut scope,
"
    print(MY_CONSTANT);     // optimized to: print(42)

    fn foo() {
        MY_CONSTANT         // optimized to: fn foo() { 42 }
    }

    print(foo());           // prints 42
")?;
```

The script will act differently when [script optimization] is disabled because script [functions]
are _pure_ and typically cannot see [constants] within the custom [`Scope`].

Therefore, constants in [functions] now throw a runtime error.

```rust,no_run
use rhai::{Engine, Scope, OptimizationLevel};

let mut engine = Engine::new();

// Turn off script optimization, no constants propagation is performed
engine.set_optimization_level(OptimizationLevel::None);

let mut scope = Scope::new();

// Add constant to custom scope
scope.push_constant("MY_CONSTANT", 42_i64);

engine.run_with_scope(&mut scope,
"
    print(MY_CONSTANT);     // prints 42

    fn foo() {
        MY_CONSTANT         // <- 'foo' cannot see 'MY_CONSTANT'
    }

    print(foo());           // error: 'MY_CONSTANT' not found
")?;
```
~~~

~~~admonish danger "Caveat: Beware of large constants"

[Constants] propagation replaces each usage of the [constant] with a clone of its value.

This may have negative implications to performance if the [constant] value is expensive to clone
(e.g. if the type is very large).

```rust,no_run
let mut scope = Scope::new();

// Push a large constant into the scope...
let big_type = AVeryLargeType::take_long_time_to_create();
scope.push_constant("MY_BIG_TYPE", big_type);

// Causes each usage of 'MY_BIG_TYPE' in the script below to be replaced
// by cloned copies of 'AVeryLargeType'.
let result = engine.run_with_scope(&mut scope,
"
    let value = MY_BIG_TYPE.value;
    let data = MY_BIG_TYPE.data;
    let len = MY_BIG_TYPE.len();
    let has_options = MY_BIG_TYPE.has_options();
    let num_options = MY_BIG_TYPE.options_len();
")?;
```

To avoid this, compile the script first to an [`AST`] _without_ the [constants], then evaluate the
[`AST`] (e.g. with `Engine::eval_ast_with_scope` or `Engine::run_ast_with_scope`) together with
the [constants].
~~~

~~~admonish danger "Caveat: Constants may be modified by Rust methods"

If the [constants] are modified later on (yes, it is possible, via Rust _methods_),
the modified values will not show up in the optimized script.
Only the initialization values of [constants] are ever retained.

```rust,no_run
const MY_SECRET_ANSWER = 42;

MY_SECRET_ANSWER.update_to(666);    // assume 'update_to(&mut i64)' is a Rust function

print(MY_SECRET_ANSWER);            // prints 42 because the constant is propagated
```

This is almost never a problem because real-world scripts seldom modify a [constant],
but the possibility is always there.
~~~
