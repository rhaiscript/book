Import a Module
===============

`import` Statement
------------------

```admonish tip.side.wide "Tip"

A [module](index.md) that is only `import`-ed but not given any name is simply run.

This is a very simple way to run another script file from within a script.
```

A [module](index.md) can be _imported_ via the `import` statement, and be given a name.

Its members can be accessed via `::` similar to C++.

```js
import "crypto_banner";         // run the script file 'crypto_banner.rhai' without creating an imported module

import "crypto" as lock;        // run the script file 'crypto.rhai' and import it as a module named 'lock'

const SECRET_NUMBER = 42;

let mod_file = `crypto_${SECRET_NUMBER}`;

import mod_file as my_mod;      // load the script file "crypto_42.rhai" and import it as a module named 'my_mod'
                                // notice that module path names can be dynamically constructed!
                                // any expression that evaluates to a string is acceptable after the 'import' keyword

lock::encrypt(secret);          // use functions defined under the module via '::'

lock::hash::sha256(key);        // sub-modules are also supported

print(lock::status);            // module variables are constants

lock::status = "off";           // <- runtime error: cannot modify a constant
```


```admonish info "Imports are _scoped_"

[Modules](index.md) imported via `import` statements are only accessible inside the relevant block scope.

~~~js
import "hacker" as h;           // import module - visible globally

if secured {                    // <- new block scope
    let mod = "crypt";

    import mod + "o" as c;      // import module (the path needs not be a constant string)

    let x = c::encrypt(key);    // use a function in the module

    h::hack(x);                 // global module 'h' is visible here
}                               // <- module 'c' disappears at the end of the block scope

h::hack(something);             // this works as 'h' is visible

c::encrypt(something);          // <- this causes a run-time error because
                                //    module 'c' is no longer available!

fn foo(something) {
    h::hack(something);         // <- this also works as 'h' is visible
}

for x in 0..1000 {
    import "crypto" as c;       // <- importing a module inside a loop is a Very Bad Idea™

    c.encrypt(something);
}
~~~
```

~~~admonish note "Place `import` statements at the top"

`import` statements can appear anywhere a normal statement can be, but in the vast majority of cases they are
usually grouped at the top (beginning) of a script for manageability and visibility.

It is not advised to deviate from this common practice unless there is a _Very Good Reason™_.

Especially, do not place an `import` statement within a loop; doing so will repeatedly re-load the
same [module](index.md) during every iteration of the loop!
~~~

~~~admonish danger "Recursive imports"

Beware of _import cycles_ &ndash; i.e. recursively loading the same [module](index.md).
This is a sure-fire way to cause a stack overflow error.

For instance, importing itself always causes an infinite recursion:

```js
┌────────────┐
│ hello.rhai │
└────────────┘

import "hello" as foo;          // import itself - infinite recursion!

foo::do_something();
```

[Modules](index.md) cross-referencing also cause infinite recursion:

```js
┌────────────┐
│ hello.rhai │
└────────────┘

import "world" as foo;
foo::do_something();


┌────────────┐
│ world.rhai │
└────────────┘

import "hello" as bar;
bar::do_something_else();
```
~~~
