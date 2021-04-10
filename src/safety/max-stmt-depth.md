Maximum Statement Depth
======================

{{#include ../links.md}}

Limit How Deeply-Nested a Statement Can Be
-----------------------------------------

Rhai by default limits statements and expressions nesting to a maximum depth of 64
(which should be plenty) when they are at _global_ level, but only a depth of 32
when they are within function bodies.

For debug builds, these limits are set further downwards to 32 and 16 respectively.

That is because it is possible to overflow the [`Engine`]'s stack when it tries to
recursively parse an extremely deeply-nested code stream.

```rust , no_run
// The following, if long enough, can easily cause stack overflow during parsing.
let a = (1+(1+(1+(1+(1+(1+(1+(1+(1+(1+(...)+1)))))))))));
```

This limit may be changed via the `Engine::set_max_expr_depths` method.

There are two limits to set, one for the maximum depth at global level, and the other for function bodies.

A script exceeding the maximum nesting depths will terminate with a parsing error.
The malicious [`AST`] will not be able to get past parsing in the first place.

This check can be disabled via the [`unchecked`] feature for higher performance (but higher risks as well).

```rust , no_run
let mut engine = Engine::new();

engine.set_max_expr_depths(50, 5);  // allow nesting up to 50 layers of expressions/statements
                                    // at global level, but only 5 inside functions
```

Beware that there may be multiple layers for a simple language construct, even though it may correspond
to only one AST node. That is because the Rhai _parser_ internally runs a recursive chain of function calls
and it is important that a malicious script does not panic the parser in the first place.


Beware of Recursion
-------------------

_Functions_ are placed under stricter limits because of the multiplicative effect of _recursion_.

A script can effectively call itself while deep inside an expression chain within the function body,
thereby overflowing the stack even when the level of recursion is within limit.

In general, make sure that `C x ( 5 + F ) + S` layered calls do not cause a stack overflow, where:

* `C` = maximum call stack depth,
* `F` = maximum statement depth for functions,
* `S` = maximum statement depth at global level.
