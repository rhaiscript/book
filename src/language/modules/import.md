Import a Module
===============

{{#include ../../links.md}}


Before a module can be used (via an `import` statement) in a script, there must be a [module resolver]
registered into the [`Engine`], the default being the `FileModuleResolver`.

See the section on [_Module Resolvers_][module resolver] for more details.


`import` Statement
-----------------

A module can be _imported_ via the `import` statement, and be given a name.
Its members can be accessed via `::` similar to C++.

A module that is only `import`-ed but not under any module name is commonly used for initialization purposes,
where the module script contains initialization statements that puts the functions registered with the
[`Engine`] into a particular state.

```rust,no_run
import "crypto_init";           // run the script file 'crypto_init.rhai' without creating an imported module

import "crypto" as lock;        // run the script file 'crypto.rhai' and import it as a module named 'lock'

const SECRET_NUMBER = 42;

let mod_file = "crypto_" + SECRET_NUMBER;

import mod_file as my_mod;      // load the script file "crypto_42.rhai" and import it as a module named 'my_mod'
                                // notice that module path names can be dynamically constructed!
                                // any expression that evaluates to a string is acceptable after the 'import' keyword

lock::encrypt(secret);          // use functions defined under the module via '::'

lock::hash::sha256(key);        // sub-modules are also supported

print(lock::status);            // module variables are constants

lock::status = "off";           // <- runtime error - cannot modify a constant
```


Scoped Imports
--------------

`import` statements are _scoped_, meaning that they are only accessible inside the scope that they're imported.

They can appear anywhere a normal statement can be, but in the vast majority of cases `import` statements are
usually grouped:

1) at the beginning of a script or,

2) at the beginning of a [function]'s body.

It is not advised to deviate from this common practice unless there is a _Very Good Reason™_.

Especially, do not place an `import` statement within a loop; doing so will repeatedly re-load the
same module during every iteration of the loop!

On the other hand, [functions] in _Rhai_ are _pure_ and they form closed environments.
They cannot access anything outside of their body, including [variables], [constants], imports etc.

In order to use a module inside a function, it must be imported within the [function] body.

```rust,no_run
import "hacker" as h;           // import module

if secured {                    // <- new block scope
    let mod = "crypt";

    import mod + "o" as c;      // import module (the path needs not be a constant string)

    let x = c::encrypt(key);    // use a function in the module

    h::hack(x);                 // use imports from parent blocks
}                               // <- module 'c' disappears at the end of the block scope

c::encrypt(others);             // <- this causes a run-time error because
                                //    module 'c' is no longer available!

fn breaks(data) {
    h::hack(data);              // <- this will cause a run-time error because
                                //    module 'h' cannot be accessed here!
}

fn works(data) {
    import "hacker" as h;       // import module
    h::hack(data);              // <- this now works
}

for x in range(0, 1000) {
    import "crypto" as c;       // <- importing a module inside a loop is a Very Bad Idea™

    c.encrypt(something);
}
```


Recursive Imports
----------------

Beware of _import cycles_ &ndash; i.e. recursively loading the same module. This is a sure-fire way to
cause a stack overflow in the [`Engine`], unless stopped by setting a limit for [maximum number of modules].

For instance, importing itself always causes an infinite recursion:

```rust,no_run
+------------+
| hello.rhai |
+------------+

import "hello" as foo;          // import itself - infinite recursion!

foo::do_something();
```

Modules cross-referencing also cause infinite recursion:

```rust,no_run
+------------+
| hello.rhai |
+------------+

import "world" as foo;
foo::do_something();


+------------+
| world.rhai |
+------------+

import "hello" as bar;
bar::do_something_else();
```
