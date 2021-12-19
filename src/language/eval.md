`eval` Function
===============

{{#include ../links.md}}

Or "How to Shoot Yourself in the Foot even Easier"
------------------------------------------------

Saving the best for last, there is the ever-dreaded... `eval` function!

```rust no_run
let x = 10;

fn foo(x) { x += 12; x }

let script = "let y = x;";      // build a script
script +=    "y += foo(y);";
script +=    "x + y";

let result = eval(script);      // <- look, JavaScript, we can also do this!

result == 42;

x == 10;                        // prints 10: functions call arguments are passed by value
y == 32;                        // prints 32: variables defined in 'eval' persist!

eval("{ let z = y }");          // to keep a variable local, use a statement block

print(z);                       // <- error: variable 'z' not found

"print(42)".eval();             // <- nope... method-call style doesn't work with 'eval'
```

Script segments passed to `eval` execute inside the current [`Scope`], so they can access and modify _everything_,
including all variables that are visible at that position in code! It is almost as if the script segments were
physically pasted in at the position of the `eval` call.


Cannot Define New Functions
--------------------------

New functions cannot be defined within an `eval` call, since functions can only be defined at the _global_ level,
not inside another function call!

```rust no_run
let script = "x += 32";
let x = 10;
eval(script);                   // variable 'x' in the current scope is visible!
print(x);                       // prints 42

// The above is equivalent to:
let script = "x += 32";
let x = 10;
x += 32;
print(x);
```


`eval` is Evil
--------------

For those who subscribe to the (very sensible) motto of ["`eval` is evil"](http://linterrors.com/js/eval-is-evil),
disable `eval` via [`Engine::disable_symbol`][disable keywords and operators].

```rust no_run
engine.disable_symbol("eval");  // disable usage of 'eval'
```
