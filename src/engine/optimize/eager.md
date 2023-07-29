Eager Function Evaluation When Using Full Optimization Level
============================================================

{{#include ../../links.md}}

When the optimization level is [`OptimizationLevel::Full`], the [`Engine`] assumes all functions to
be _pure_ and will _eagerly_ evaluated all function calls with constant arguments, using the result
to replace the call.

This also applies to all operators (which are implemented as functions).

```rust
// When compiling the following with OptimizationLevel::Full...

const DECISION = 1;
                            //    this condition is now eliminated because 'sign(DECISION) > 0'
if sign(DECISION) > 0       // <- is a call to the 'sign' and '>' functions, and they return 'true'
{
    print("hello!");        // <- this block is promoted to the parent level
} else {
    print("boo!");          // <- this block is eliminated because it is never reached
}

print("hello!");            // <- the above is equivalent to this
                            //    ('print' and 'debug' are handled specially)
```

~~~admonish danger "Won't this be dangerous?"

Yes! _Very!_

```rust
// Nuclear silo control
if launch_nukes && president_okeyed {
    print("This is NOT a drill!");
    update_defcon(1);
    start_world_war(3);
    launch_all_nukes();
} else {
    print("This is a drill.  Thank you for your cooperation.");
}
```

In the script above (well... as if nuclear silos will one day be controlled by Rhai scripts),
the functions `update_defcon`, `start_world_war` and `launch_all_nukes` will be evaluated
during _compilation_ because they have constant arguments.

The [variables] `launch_nukes` and `president_okeyed` are never checked, because the script
actually has not yet been run!  The functions are called during compilation.
This is, _obviously_, not what you want.

**Moral of the story: compile with an [`Engine`] that does not have any functions registered.
Register functions _AFTER_ compilation.**
~~~

~~~admonish question "Why would I ever want to do this then?"

Good question! There are two reasons:

* A function call may result in cleaner code than the resultant value.
  In Rust, this would have been handled via a `const` function.

* Evaluating a value to a [custom type] that has no representation in script.

```rust
// A complex function that returns a unique ID based on the arguments
let id = make_unique_id(123, "hello", true);

// The above is arguably clearer than:
//   let id = 835781293546; // generated from 123, "hello" and true

// A custom type that cannot be represented in script
let complex_obj = make_complex_obj(42);
```
~~~