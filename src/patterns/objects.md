Objects with Defined Behaviors
==============================

{{#include ../links.md}}


```admonish info "Usage scenario"

* To simulate specific _object types_ coded in Rust with fields and methods.

* Native Rust methods (not scripted) are pre-defined for these types.
```

```admonish abstract "Key concepts"

* Use an [object map] as the main container of the type.

* Create [function pointers] binding to _native Rust functions_ and store them as properties
  of the [object map].
```

Rhai does not have _objects_ per se and is not object-oriented (in the traditional sense), but it is
possible to _simulate_ object-oriented programming via [object maps].

When using [object maps] to simulate objects (See [here](oop.md) for more details), a property that
holds a [function pointer] can be called like a method function with the [object map] being the
object of the method call.

It is also possible to create [function pointers] that bind to native Rust functions/closures so
that those are called when the [function pointers] are called.


Implementation
--------------

A [function pointer] can be created that binds to a specific Rust function or closure.

(See [here]({{rootUrl}}/language/fn-ptr.md#bind-to-a-native-rust-function) for details on using
`FnPtr::from_fn` and `FnPtr::from_dyn_fn`).

```rust
// This is the pre-defined behavior for the 'Awesome' object type.
fn my_awesome_fn(ctx: NativeCallContext, args: &mut[&mut Dynamic]) -> Result<Dynamic, Box<EvalAltResult>> {
    // Check number of arguments
    if args.len() != 2 {
        return Err("one argument is required, plus the object".into());
    }

    // Get call arguments
    let x = args[1].try_cast::<i64>().map_err(|_| "argument must be an integer".into())?;

    // Get mutable reference to the object map, which is passed as the first argument
    let map = &mut *args[0].as_map_mut().map_err(|_| "object must be a map".into())?;

    // Do something awesome here ...
    let result = ...

    Ok(result.into())
}

// Register a function to create a pre-defined object
engine.register_fn("create_awesome_object", || {
    // Use an object map as base
    let mut map = Map::new();

    // Create a function pointer that binds to 'my_awesome_fn'
    let fp = FnPtr::from_fn("awesome", my_awesome_fn)?;
    //                      ^ name of method
    //                                 ^ native function

    // Store the function pointer in the object map
    map.insert("awesome".into(), fp.into());

    Ok(Dynamic::from_map(map))
});
```

The [object map] can then be used just like any object-oriented object.

```rust
let obj = create_awesome_object();

let result = obj.awesome(42);   // calls 'my_awesome_fn' with '42' as argument
```
