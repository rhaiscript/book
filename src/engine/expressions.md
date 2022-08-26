Evaluate Expressions Only
=========================

{{#include ../links.md}}

~~~admonish tip.side "Tip: `Dynamic`"

Use [`Dynamic`] if you're uncertain of the return type.
~~~

Very often, a use case does not require a full-blown scripting _language_, but only needs to
evaluate _expressions_.

In these cases, use the `Engine::compile_expression` and `Engine::eval_expression` methods or their
`_with_scope` variants.

```rust
let result: i64 = engine.eval_expression("2 + (10 + 10) * 2")?;

let result: Dynamic = engine.eval_expression("get_value(42)")?;

// Usually this is done together with a custom scope with variables...

let mut scope = Scope::new();

scope.push("x", 42_i64);
scope.push_constant("SCALE", 10_i64);

let result: i64 = engine.eval_expression_with_scope(&mut scope,
                        "(x + 1) * SCALE"
                  )?;
```

~~~admonish bug "No statements allowed"

When evaluating _expressions_, no full-blown statement (e.g. [`while`], [`for`], `fn`) &ndash;
not even [variable] assignment &ndash; is supported and will be considered syntax errors.

This is true also for [statement expressions]({{rootUrl}}/language/statement-expression.md)
and [anonymous functions]/[closures].

```rust
// The following are all syntax errors because the script
// is not a strict expression.

engine.eval_expression::<()>("x = 42")?;

let ast = engine.compile_expression("let x = 42")?;

let result = engine.eval_expression_with_scope::<i64>(&mut scope,
                    "{ let y = calc(x); x + y }"
             )?;

let fp: FnPtr = engine.eval_expression("|x| x + 1")?;
```
~~~


~~~admonish tip "Tip: `if`-expressions and `switch`-expressions"

[`if` expressions]({{rootUrl}}/language/if-expression.md) are allowed if both statement blocks
contain only a single expression each.

[`switch` expressions]({{rootUrl}}/language/switch-expression.md) are allowed if all match
actions are expressions and not statements.

```rust
// The following are allowed.

let result = engine.eval_expression_with_scope::<i64>(&mut scope,
                    "if x { 42 } else { 123 }"
             )?;

let result = engine.eval_expression_with_scope::<i64>(&mut scope, "
                    switch x {
                        0 => x * 42,
                        1..=9 => foo(123) + bar(1),
                        10 => 0,
                    }
             ")?;
```
~~~
