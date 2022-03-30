Dead Code Elimination
=====================

{{#include ../../links.md}}

```admonish question.side.wide "Who writes dead code?"

Nobody deliberately writes scripts with dead code.

They are, however, extremely common in template-based machine-generated scripts.
```

Rhai attempts to eliminate _dead code_.

"Dead code" is code that does nothing and has no side effects.

Example is an pure expression by itself as a statement (allowed in Rhai).
The result of the expression is calculated then immediately discarded and not used.

```rust
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

```rust
{
    let x = 999;
    foo(42);
    666
}
```
