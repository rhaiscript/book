Scriptable Event Handler with State<br/>Main Style
=================================================

{{#include ../links.md}}


Example
-------

A runnable example of this implementation is included.

See the [_Examples_]({{rootUrl}}/start/examples/index.md) section for details.


Initialize Handler Instance with `Engine::call_fn_raw`
----------------------------------------------------

Use `Engine::call_fn_raw` instead of `Engine::call_fn` in order to retain new [variables] defined
inside the custom [`Scope`] when running the `init` function.

```rust,no_run
impl Handler {
    // Create a new 'Handler'.
    pub fn new(path: impl Into<PathBuf>) -> Self {
        let mut engine = Engine::new();

                    :
            // Code omitted
                    :

        // Run the 'init' function to initialize the state, retaining variables.
        // In a real application you'd again be handling errors...
        engine.call_fn_raw(&mut scope, &ast, false, false, "init", None, []).unwrap();
        //                                          ^^^^^ do not rewind scope

                    :
            // Code omitted
                    :

        Self { engine, scope, ast }
    }
}
```


Handler Scripting Style
-----------------------

Because the stored state is kept in a custom [`Scope`], it is possible for all [functions] defined
in the handler script to access and modify these state variables.

The API registered with the [`Engine`] can be also used throughout the script.

### Sample script

```js
/// Initialize user-provided state (shadows system-provided state, if any).
/// Because 'call_fn_raw' is used to call this, new variables introduced
/// will remain inside the custom 'Scope'.
fn init() {
    // Add 'bool_state' and 'obj_state' as new state variables
    let bool_state = false;
    let obj_state = new_state(0);

    // Constants can also be added!
    const EXTRA_CONSTANT = "hello, world!";
}

/// Without 'OOP' support, the can only be a function.
fn log(value, data) {
    print(`State = ${value}, data = ${data}`);
}

/// 'start' event handler
fn start(data) {
    if bool_state {
        throw "Already started!";
    }
    if obj_state.func1() || obj_state.func2() {
        throw "Conditions not yet ready to start!";
    }
    bool_state = true;
    obj_state.value = data;

    // Constants 'MY_CONSTANT' and 'EXTRA_CONSTANT'
    // in custom scope are also visible!
    print(`MY_CONSTANT = ${MY_CONSTANT}`);
    print(`EXTRA_CONSTANT = ${EXTRA_CONSTANT}`);
}

/// 'end' event handler
fn end(data) {
    if !bool_state {
        throw "Not yet started!";
    }
    if !obj_state.func1() && !obj_state.func2() {
        throw "Conditions not yet ready to end!";
    }
    bool_state = false;
    obj_state.value = data;
}

/// 'update' event handler
fn update(data) {
    obj_state.value += process(data);

    // Without OOP support, can only call function
    log(obj_state.value, data);
}
```


Disadvantages of This Style
---------------------------

This style is simple to implement and intuitive to use, but it is not very flexible.

New user state [variables] are introduced by evaluating a special initialization script [function]
(e.g. `init`) that defines them, and `Engine::call_fn_raw` is used to keep them inside the custom [`Scope`].

However, this has the disadvantage that no other [function] can introduce new state [variables],
otherwise they'd simply _shadow_ existing [variables] in the custom [`Scope`].  Thus, [functions] are
called during events via `Engine::call_fn` which does not retain any [variables].

When there are a large number of state [variables], this style also makes it easy for local [variables]
defined in user [functions] to accidentally _shadow_ a state [variable] with a [variable] that just
happens to be the same name.

```rust,no_run
// 'start' event handler
fn start(data) {
    let bool_state = false; // <- oops! bad variable name!
        
        :                   // there is now no way to access the
        :                   // state variable 'bool_state'...

    if bool_state {         // <- 'bool_state' is not the right thing
        ...                 //    unless this is what you actually want
    }

        :
        :
}
```
