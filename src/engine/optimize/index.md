Script Optimization
===================

{{#include ../../links.md}}

Rhai includes an _optimizer_ that tries to optimize a script after parsing.
This can reduce resource utilization and increase execution speed.

Script optimization can be turned off via the [`no_optimize`] feature.


Dead Code Removal
----------------

Rhai attempts to eliminate _dead code_ (i.e. code that does nothing, for example an expression by
itself as a statement, which is allowed in Rhai).

```rust no_run
{
    let x = 999;            // NOT eliminated: variable may be used later on (perhaps even an 'eval')
    
    123;                    // eliminated: no effect
    
    "hello";                // eliminated: no effect
    
    [1, 2, x, 4];           // eliminated: no effect
    
    if 42 > 0 {             // '42 > 0' is replaced by 'true' and the first branch promoted
        foo(42);            // promoted, NOT eliminated: the function 'foo' may have side-effects
    } else {
        bar(x);             // eliminated: branch is never reached
    }
    
    let z = x;              // eliminated: local variable, no side-effects, and only pure afterwards
    
    666                     // NOT eliminated: this is the return value of the block,
                            // and the block is the last one so this is the return value of the whole script
}
```

The above script optimizes to:

```rust no_run
{
    let x = 999;
    foo(42);
    666
}
```

Normally, nobody deliberately writes scripts with dead code, but it is extremely common for
template-based machine-generated scripts, especially where [constants] are involved.


Constants Propagation
--------------------

[Constants] propagation is used to remove dead code:

```rust no_run
const ABC = true;

if ABC || some_work() { print("done!"); }   // 'ABC' is constant so it is replaced by 'true'...

if true || some_work() { print("done!"); }  // since '||' short-circuits, 'some_work' is never called

if true { print("done!"); }                 // <- the line above is equivalent to this

print("done!");                             // <- the line above is further simplified to this
                                            //    because the condition is always true
```

These are quite effective for template-based machine-generated scripts where certain [constant] values
are spliced into the script text in order to turn on/off certain sections.

For fixed script texts, the [constant] values can also be provided in a custom [`Scope`] object to
the [`Engine`] for use in compilation and evaluation.

```rust no_run
use rhai::{Engine, Scope};

let engine = Engine::new();

let mut scope = Scope::new();

// Add constant to custom scope
scope.push_constant("ABC", true);

// Evaluate script with custom scope
engine.run_with_scope(&mut scope,
"
    if ABC {    // 'ABC' is replaced by 'true'
        print("done!");
    }
")?;
```

### Caveat &ndash; constants in custom scope are also propagated into functions

[Constants] defined at _global_ level typically cannot be seen by script [functions] because they are _pure_.

```rust no_run
const MY_CONSTANT = 42;     // <- constant defined at global level

print(MY_CONSTANT);         // <- optimized to: print(42)

fn foo() {
    MY_CONSTANT             // <- not optimized: 'foo' cannot see 'MY_CONSTANT'
}

print(foo());               // error: 'MY_CONSTANT' not found
```

When [constants] are provided in a custom [`Scope`] (e.g. via `Engine::compile_with_scope`,
`Engine::eval_with_scope` or `Engine::run_with_scope`) instead of defined within the same script,
they are also propagated to [functions].

This is usually the intuitive usage and behavior expected by regular users, even though it means
that a script will behave differently (essentially a runtime error) when [script optimization] is disabled.

```rust no_run
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

```rust no_run
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

### Caveat &ndash; beware large constants

[Constants] propagation replaces each usage of the [constant] with a clone of its value.

This may have negative implications to performance if the [constant] value is expensive to clone
(e.g. if the type is very large).

```rust no_run
let mut scope = Scope::new();

// Push a large constant into the scope...
scope.push_constant("MY_BIG_TYPE", AVeryLargeType::take_long_time_to_create());

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

### Caveat &ndash; constants may be modified by Rust methods

If the [constants] are modified later on (yes, it is possible, via Rust _methods_),
the modified values will not show up in the optimized script.
Only the initialization values of [constants] are ever retained.

```rust no_run
const MY_SECRET_ANSWER = 42;

MY_SECRET_ANSWER.update_to(666);    // assume 'update_to(&mut i64)' is a Rust function

print(MY_SECRET_ANSWER);            // prints 42 because the constant is propagated
```

This is almost never a problem because real-world scripts seldom modify a [constant],
but the possibility is always there.


Op-Assignment Rewrite
---------------------

Usually, an _op-assignment_ operator (e.g. `+=` for append) takes a mutable first parameter
(i.e. `&mut`) while the corresponding simple operator (i.e. `+`) does not.

This has huge performance implications because arguments passed as reference are always cloned.

```rust no_run
let big = create_some_very_big_type();

big = big + 1;
//    ^ 'big' is cloned here

// The above is equivalent to:
let temp_value = big + 1;
big = temp_value;

big += 1;           // <- 'big' is NOT cloned
```

The script optimizer rewrites normal expressions into _op-assignment_ style wherever possible.

However, and only those involving **simple variable references** are optimized.
In other words, no _common sub-expression elimination_ is performed by Rhai.

```rust no_run
x = x + 1;          // <- this statement...

x += 1;             // ... is rewritten as this

x[y] = x[y] + 1;    // <- but this is not, so this is MUCH slower...

x[y] + 1;           // ... than this
```


Eager Operator Evaluation
------------------------

Beware, however, that most operators are actually function calls, and those functions can be overridden,
so whether they are optimized away depends on the situation:

* If the operands are not [constant] values, it is not optimized.

* If the operator is [overloaded][operator overloading], it is not optimized because the overloading
  function may not be _pure_ (i.e. may cause side-effects when called).

* If the operator is not _built-in_ (see list of [built-in operators]), it is not optimized.

* If the operator is a [built-in operator] for a [standard type][standard types],
  it is called and replaced by a [constant] result.

Rhai guarantees that no external function will be run (in order not to trigger side-effects) during the
optimization process (unless the optimization level is set to [`OptimizationLevel::Full`]).

```js , no_run
// The following is most likely generated by machine.

const DECISION = 1;             // this is an integer, one of the standard types

if DECISION == 1 {              // this is optimized into 'true'
    :
} else if DECISION == 2 {       // this is optimized into 'false'
    :
} else if DECISION == 3 {       // this is optimized into 'false'
    :
} else {
    :
}

// Or an equivalent using 'switch':

switch DECISION {
    1 => ...,                   // this statement is promoted
    2 => ...,                   // this statement is eliminated
    3 => ...,                   // this statement is eliminated
    _ => ...                    // this statement is eliminated
}
```

Because of the eager evaluation of [operators][built-in operators] for [standard types], many
[constant] expressions will be evaluated and replaced by the result.

```rust no_run
let x = (1+2)*3-4/5%6;          // will be replaced by 'let x = 9'

let y = (1 > 2) || (3 <= 4);    // will be replaced by 'let y = true'
```

For operators that are not optimized away due to one of the above reasons, the function calls
are simply left behind:

```rust no_run
// Assume 'new_state' returns some custom type that is NOT one of the standard types.
// Also assume that the '==' operator is defined for that custom type.
const DECISION_1 = new_state(1);
const DECISION_2 = new_state(2);
const DECISION_3 = new_state(3);

if DECISION == 1 {              // NOT optimized away because the operator is not built-in
    :                           // and may cause side-effects if called!
    :
} else if DECISION == 2 {       // same here, NOT optimized away
    :
} else if DECISION == 3 {       // same here, NOT optimized away
    :
} else {
    :
}
```

Alternatively, turn the optimizer to [`OptimizationLevel::Full`].
