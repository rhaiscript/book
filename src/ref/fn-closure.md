Simulating Closures &ndash; Capture External Variables via Automatic Currying
=============================================================================

~~~admonish tip.side "Tip: `is_shared`"

Use `is_shared` to check whether a particular dynamic value is shared.
~~~

Since [anonymous functions](fn-anon.md) de-sugar to standard function definitions, they retain all
the behaviors of Rhai functions, including being _pure_, having no access to external
[variables](variables.md).

The [anonymous function](fn-anon.md) syntax, however, automatically _captures_
[variables](variables.md) that are not defined within the current scope, but are defined in the
external scope &ndash; i.e. the scope where the [anonymous function](fn-anon.md) is created.

[Variables](variables.md) that are accessible during the time the [anonymous function](fn-anon.md)
is created can be captured, as long as they are not shadowed by local [variables](variables.md)
defined within the function's scope.

The captured [variables](variables.md) are automatically converted into **reference-counted shared values**.

Therefore, similar to closures in many languages, these captured shared values persist through
reference counting, and may be read or modified even after the [variables](variables.md) that hold
them go out of scope and no longer exist.

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


~~~admonish warning "Beware: Captured variables are truly shared"

The example below is a typical tutorial sample for many languages to illustrate the traps
that may accompany capturing external [variables](variables.md) in closures.

It prints `9`, `9`, `9`, ... `9`, `9`, not `0`, `1`, `2`, ... `8`, `9`, because there is ever only
_one_ captured [variable](variables.md), and all ten closures capture the _same_
[variable](variables.md).

```rust
let list = [];

for i in 0..10 {
    list.push(|| print(i));     // the for loop variable 'i' is captured
}

list.len() == 10;               // 10 closures stored in the array

list[0].type_of() == "Fn";      // make sure these are closures

for f in list {
    f.call();                   // all references to 'i' point to the same variable!
}
```
~~~

~~~admonish danger "Prevent data races"

Data races are possible in Rhai scripts.

Avoid performing a method call on a captured shared [variable](variables.md) (which essentially
takes a mutable reference to the shared object) while using that same [variable](variables.md) as a
parameter in the method call &ndash; this is a sure-fire way to generate a data race error.

If a shared value is used as the `this` pointer in a method call to a closure function,
then the same shared value _must not_ be captured inside that function, or a data race
will occur and the script will terminate with an error.

```rust
let x = 20;

x.is_shared() == false;         // 'x' not shared, so no data races

let f = |a| this += x + a;      // 'x' is captured in this closure

x.is_shared() == true;          // now 'x' is shared

x.call(f, 2);                   // <- error: data race detected on 'x'
```
~~~
