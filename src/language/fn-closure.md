Simulating Closures
===================

{{#include ../links.md}}

Capture External Variables via Automatic Currying
-------------------------------------------------

~~~admonish tip.side "Tip: `is_shared`"

Use `Dynamic::is_shared` to check whether a particular [`Dynamic`] value is shared.
~~~

Since [anonymous functions] de-sugar to standard function definitions, they retain all the behaviors
of Rhai functions, including being _pure_, having no access to external [variables].

The [anonymous function] syntax, however, automatically _captures_ [variables] that are not defined
within the current scope, but are defined in the external scope &ndash; i.e. the scope where the
[anonymous function] is created.

[Variables] that are accessible during the time the [anonymous function] is created can be captured,
as long as they are not shadowed by local [variables] defined within the function's scope.

The captured [variables] are automatically converted into **reference-counted shared values**
(`Rc<RefCell<Dynamic>>`, or `Arc<RwLock<Dynamic>>` under [`sync`]).

Therefore, similar to closures in many languages, these captured shared values persist through
reference counting, and may be read or modified even after the [variables] that hold them go out of
scope and no longer exist.

```admonish tip.small "Tip: Disable closures"

[Automatic currying] can be turned off via the [`no_closure`] feature.
```


Examples
--------

```rust
let x = 1;                          // a normal variable

x.is_shared() == false;

let f = |y| x + y;                  // variable 'x' is auto-curried (captured) into 'f'

x.is_shared() == true;              // 'x' is now a shared value!

f.call(2) == 3;                     // 1 + 2 == 3

x = 40;                             // changing 'x'...

f.call(2) == 42;                    // the value of 'x' is 40 because 'x' is shared

// The above de-sugars into something like this:

fn anon_0001(x, y) { x + y }        // parameter 'x' is inserted

make_shared(x);                     // convert variable 'x' into a shared value

let f = Fn("anon_0001").curry(x);   // shared 'x' is curried
```


Beware: Captured Variables are Truly Shared
-------------------------------------------

The example below is a typical tutorial sample for many languages to illustrate the traps
that may accompany capturing external [variables] in closures.

It prints `9`, `9`, `9`, ... `9`, `9`, not `0`, `1`, `2`, ... `8`, `9`, because there is
ever only _one_ captured [variable], and all ten closures capture the _same_ [variable].

```rust
let list = [];

for i in 0..10 {
    list.push(|| print(i));         // the for loop variable 'i' is captured
}

list.len() == 10;                   // 10 closures stored in the array

list[0].type_of() == "Fn";          // make sure these are closures

for f in list {
    f.call();                       // all references to 'i' are the same variable!
}
```


Therefore &ndash; Be Careful to Prevent Data Races
--------------------------------------------------

Rust does not have data races, but that doesn't mean Rhai doesn't.

Avoid performing a method call on a captured shared [variable] (which essentially takes a
mutable reference to the shared object) while using that same [variable] as a parameter
in the method call &ndash; this is a sure-fire way to generate a data race error.

If a shared value is used as the `this` pointer in a method call to a closure function,
then the same shared value _must not_ be captured inside that function, or a data race
will occur and the script will terminate with an error.

```rust
let x = 20;

x.is_shared() == false;             // 'x' is not shared, so no data race is possible

let f = |a| this += x + a;          // 'x' is captured in this closure

x.is_shared() == true;              // now 'x' is shared

x.call(f, 2);                       // <- error: data race detected on 'x'
```


Data Races in `sync` Builds Can Become Deadlocks
------------------------------------------------

Under the [`sync`] feature, shared values are guarded with a `RwLock`, meaning that data race
conditions no longer raise an error.

Instead, they wait endlessly for the `RwLock` to be freed, and thus can become deadlocks.

On the other hand, since the same thread (i.e. the [`Engine`] thread) that is holding the lock
is attempting to read it again, this may also [panic](https://doc.rust-lang.org/std/sync/struct.RwLock.html#panics-1)
depending on the O/S.

```rust
let x = 20;

let f = |a| this += x + a;          // 'x' is captured in this closure

// Under `sync`, the following may wait forever, or may panic,
// because 'x' is locked as the `this` pointer but also accessed
// via a captured shared value.
x.call(f, 2);
```


TL;DR
-----

```admonish question "How is it actually implemented?"

The actual implementation of closures de-sugars to:

1. Keeping track of what [variables] are accessed inside the [anonymous function],

2. If a [variable] is not defined within the [anonymous function's][anonymous function] scope,
   it is looked up _outside_ the [function] and in the current execution scope &ndash;
   where the [anonymous function] is created.

3. The [variable] is added to the parameters list of the [anonymous function], at the front.

4. The [variable] is then converted into a **reference-counted shared value**.

   An [anonymous function] which captures an external [variable] is the only way to create a
   reference-counted shared value in Rhai.

5. The shared value is then [curried][currying] into the [function pointer] itself,
   essentially carrying a reference to that shared value and inserting it into future calls of the [function].

   This process is called [_Automatic Currying_][automatic currying], and is the mechanism through
   which Rhai simulates normal closures.
```

```admonish question "Why automatic currying?"

In concept, a closure _closes_ over captured variables from the outer scope &ndash; that's why
they are called _closures_.  When this happen, a typical language implementation hoists
those variables that are captured away from the stack frame and into heap-allocated storage.
This is because those variables may be needed after the stack frame goes away.

These heap-allocated captured variables only go away when all the closures that need them
are finished with them.  A garbage collector makes this trivial to implement &ndash; they are
automatically collected as soon as all closures needing them are destroyed.

In Rust, this can be done by reference counting instead, with the potential pitfall of creating
reference loops that will prevent those variables from being deallocated forever.
Rhai avoids this by clone-copying most data values, so reference loops are hard to create.

Rhai does the hoisting of captured variables into the heap by converting those values
into reference-counted locked values, also allocated on the heap.  The process is identical.

Closures are usually implemented as a data structure containing two items:

1. A function pointer to the function body of the closure,
2. A data structure containing references to the captured shared variables on the heap.

Usually a language implementation passes the structure containing references to captured
shared variables into the function pointer, the function body taking this data structure
as an additional parameter.

This is essentially what Rhai does, except that Rhai passes each variable individually
as separate parameters to the function, instead of creating a structure and passing that
structure as a single parameter.  This is the only difference.

Therefore, in most languages, essentially all closures are implemented as automatic currying of
shared variables hoisted into the heap, automatically passing those variables as parameters into
the function. Rhai just brings this directly up to the front.
```
