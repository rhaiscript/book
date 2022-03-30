Maximum Expression Nesting Depth
===============================

{{#include ../links.md}}

Rhai by default limits statement and expression nesting to a maximum depth of 64
(which should be plenty) when they are at _global_ level, but only a depth of 32
when they are within [function] bodies.

For debug builds, these limits are set further downwards to 32 and 16 respectively.

That is because it is possible to overflow the [`Engine`]'s stack when it tries to
recursively parse an extremely deeply-nested code stream.

```rust
// The following, if long enough, can easily cause stack overflow during parsing.
let a = (1+(1+(1+(1+(1+(1+(1+(1+(1+(1+(...)+1)))))))))));
```

This limit may be changed via [`Engine::set_max_expr_depths`][options].

There are two limits to set, one for the maximum depth at global level, and the other for function bodies.

A script exceeding the maximum nesting depths will terminate with a parse error.
The malicious [`AST`] will not be able to get past parsing in the first place.

This check can be disabled via the [`unchecked`] feature for higher performance (but higher risks as well).

```rust
let mut engine = Engine::new();

engine.set_max_expr_depths(50, 5);  // allow nesting up to 50 layers of expressions/statements
                                    // at global level, but only 5 inside functions
```

```admonish warning
Multiple layers of expressions may be generated for a simple language construct, even though it may correspond
to only one AST node.

That is because the Rhai _parser_ internally runs a recursive chain of function calls and it is important that
a malicious script does not panic the parser in the first place.
```

~~~admonish danger "Beware of recursion"

_[Functions]_ are placed under stricter limits because of the multiplicative effect of _recursion_.

A [function] can effectively call itself while deep inside an expression chain within the [function]
body, thereby overflowing the stack even when the level of recursion is within limit.

```rust
fn deep_calc(a, n) {
    (a+(a+(a+(a+(a+(a+(a+(a+(a+ ... (a+deep_calc(a,n+1)) ... )))))))))
    //                                 ^^^^^^^^^^^^^^^^ recursive call!
}

let a = 42;

let result = (a+(a+(a+(a+(a+(a+(a+(a+(a+ ... (a+deep_calc(a,0)) ... )))))))));
```

In the contrived example above, each recursive call to the function `deep_calc` adds the total
number of nested expression layers to Rhai's evaluation stack.  Sooner or later (most likely sooner
than the limit for [maximum depth of function calls][maximum call stack depth] is reached), a stack
overflow can be expected.

In general, make sure that `C x ( 5 + F ) + S` layered calls do not cause a stack overflow, where:

* `C` = maximum call stack depth,
* `F` = maximum statement depth for [functions],
* `S` = maximum statement depth at global level.
~~~
