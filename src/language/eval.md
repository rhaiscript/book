`eval` Function
===============

{{#include ../links.md}}

Or "How to Shoot Yourself in the Foot even Easier"
--------------------------------------------------

Saving the best for last, there is the ever-dreaded... `eval` [function]!

```rust
let x = 10;

fn foo(x) { x += 12; x }

let script =
"
    let y = x;
    y += foo(y);
    x + y
";

let result = eval(script);      // <- look, JavaScript, we can also do this!

result == 42;

x == 10;                        // prints 10 - arguments are passed by value
y == 32;                        // prints 32 - variables defined in 'eval' persist!

eval("{ let z = y }");          // to keep a variable local, use a statements block

print(z);                       // <- error: variable 'z' not found

"print(42)".eval();             // <- nope... method-call style doesn't work with 'eval'
```

~~~admonish danger.small "`eval` executes inside the current scope!"

Script segments passed to `eval` execute inside the _current_ [`Scope`], so they can access and modify
_everything_, including all [variables] that are visible at that position in code!

```rust
let script = "x += 32";

let x = 10;
eval(script);       // variable 'x' is visible!
print(x);           // prints 42

// The above is equivalent to:
let script = "x += 32";
let x = 10;
x += 32;
print(x);
```

`eval` can also be used to define new [variables] and do other things normally forbidden inside
a [function] call.

```rust
let script = "let x = 42";
eval(script);
print(x);           // prints 42
```

Treat it as if the script segments are physically pasted in at the position of the `eval` call.
~~~

~~~admonish warning.small "Cannot define new functions"

New [functions] cannot be defined within an `eval` call, since [functions] can only be defined at
the _global_ level!
~~~

~~~admonish failure.small "`eval` is evil"

For those who subscribe to the (very sensible) motto of ["`eval` is evil"](http://linterrors.com/js/eval-is-evil),
disable `eval` via [`Engine::disable_symbol`][disable keywords and operators].

```rust
// Disable usage of 'eval'
engine.disable_symbol("eval");
```
~~~


TL;DR
-----

~~~admonish question "Do you regret implementing `eval` in Rhai?"

Or course we do.

Having the possibility of an `eval` call disrupts any predictability in the Rhai script,
thus disabling a large number of optimizations.
~~~

```admonish question "Why did it then???!!!"

Brendan Eich puts it well: "it is just too easy to implement." _(source wanted)_
```
