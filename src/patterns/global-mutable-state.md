Mutable Global State
====================

{{#include ../links.md}}


Don't Do It™
------------

```admonish question.side "Consider JavaScript"

Generations of programmers struggled to get around mutable global state (a.k.a. the `window` object)
in the design of JavaScript.
```

In contrast to [global constants](constants.md), _mutable_ global states are **strongly
discouraged** because:

1) It is a sure-fire way to create race conditions &ndash; that is why Rust does not support it;

2) It adds considerably to debug complexity &ndash; it is difficult to reason, in large code bases,
   where/when a state value is being modified;

3) It forces hard (but obscure) dependencies between separate pieces of code that are difficult to
   break when the need arises;

4) It is almost impossible to add new layers of redirection and/or abstraction afterwards without
   major surgery.


Alternative &ndash; Use `this`
------------------------------

In the majority of the such scenarios, there is only _one_ mutable global state of interest.

Therefore, it is a _much_ better solution to bind that global state to the `this` pointer.

```rust
// Say this is a mutable global state...
let state = #{ counter: 0 };

// This function tries to access the global 'state'
// which will fail.
fn inc() {
    state.counter += 1;
}

// The function should be written with 'this'
fn inc() {
    this.counter += 1;
}

state.inc();        // call 'inc' with 'state' bound to 'this'

// Or this way... why hard-code the state in the first place?
fn inc() {
    this += 1;
}

state.counter.inc();
```

```admonish question.small "Why is this better?"

There are good reasons why using `this` is a better solution:

* the state is never _hidden_ &ndash; it is always clear to see what is being modified
* it is just as fast &ndash; the `this` pointer works by reference
* you can pass other states in, in the future, without changing the script code
* there are no hard links within functions that will be difficult to unravel
* only the [variable] bound to `this` is ever modified; everything else is immutable
```

```admonish danger.small "I don't care! I want it! Just tell me how to do it! Now!"

This is not something that Rhai encourages.  _You Have Been Warned™_.

There are two ways...
```


Option 1 &ndash; Get/Set Functions
----------------------------------

This is similar to the [Control Layer](control.md) pattern.

Use get/set functions to read/write the global mutable state.

```rust
// The globally mutable shared value
let value = Rc::new(RefCell::new(42));

// Register an API to access the globally mutable shared value
let v = value.clone();
engine.register_fn("get_global_value", move || *v.borrow());

let v = value.clone();
engine.register_fn("set_global_value", move |value: i64| *v.borrow_mut() = value);
```

These functions can be used in script [functions] to access the shared global state.

```rust
fn foo() {
    let current = get_global_value();       // Get global state value
    current += 1;
    set_global_value(current);              // Modify global state value
}
```

This option is preferred because it is possible to modify the get/set functions later on to
add/change functionalities without introducing breaking script changes.


Option 2 &ndash; Variable Resolver
----------------------------------

Declare a [variable resolver] that returns a _shared_ value which is the global state.

```rust
// Use a shared value as the global state
let value: Dynamic = 1.into();
let mut value = value.into_shared();        // convert into shared value

// Clone the shared value
let v = value.clone();

// Register a variable resolver.
engine.on_var(move |name, _, _| {
    match name
        "value" => Ok(Some(v.clone())),
        _ => Ok(None)
    }
});

// The shared global state can be modified
*value.write_lock::<i64>().unwrap() = 42;
```

The global state variable can now be used just like a normal local variable,
including modifications.

```rust
fn foo() {
    value = value * 2;
//          ^ global variable can be read
//  ^ global variable can also be modified
}
```

```admonish danger.small "Anti-Pattern"

This option makes mutable global state so easy to implement that it should actually be
considered an _Anti-Pattern_.
```
