Eager Function Evaluation When Using Full Optimization Level
==========================================================

{{#include ../../links.md}}

When the optimization level is [`OptimizationLevel::Full`], the [`Engine`] assumes all functions to
be _pure_ and will _eagerly_ evaluated all function calls with constant arguments, using the result
to replace the call.

This also applies to all operators (which are implemented as functions).

```rust,no_run
// When compiling the following with OptimizationLevel::Full...

const DECISION = 1;
                            // this condition is now eliminated because 'sign(DECISION) > 0'
if DECISION.sign() > 0 {    // is a call to the 'sign' and '>' functions, and they return 'true'
    print("hello!");        // this block is promoted to the parent level
} else {
    print("boo!");          // this block is eliminated because it is never reached
}

print("hello!");            // <- the above is equivalent to this
                            //    ('print' and 'debug' are handled specially)
```
