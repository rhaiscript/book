Variables
=========

{{#include ../links.md}}


Valid Names
-----------

```admonish tip.side-wide "Tip: Unicode Standard Annex #31 identifiers"

The [`unicode-xid-ident`] feature expands the allowed characters for variable names to the set defined by
[Unicode Standard Annex #31](http://www.unicode.org/reports/tr31/).
```

Variables in Rhai follow normal C naming rules &ndash; must contain only ASCII letters, digits and underscores `_`.

| Character set | Description              |
| :-----------: | ------------------------ |
|  `A` ... `Z`  | Upper-case ASCII letters |
|  `a` ... `z`  | Lower-case ASCII letters |
|  `0` ... `9`  | Digit characters         |
|      `_`      | Underscore character     |

However, unlike Rust, a variable name must also contain at least one ASCII letter, and an ASCII
letter must come _before_ any digits. In other words, the first character that is not an underscore `_`
must be an ASCII letter and not a digit.

```admonish question.side-wide "Why this restriction?"

To reduce confusion (and subtle bugs) because, for instance, `_1` can easily be misread (or mistyped)
as `-1`.

Rhai is dynamic without type checking, so there is no compiler to catch these typos.
```

Therefore, some names acceptable to Rust, like `_`, `_42foo`, `_1` etc., are not valid in Rhai.

For example: `c3po` and `_r2d2_` are valid variable names, but `3abc` and `____49steps` are not.

Variable names are case _sensitive_.

Variable names also cannot be the same as a [keyword] (active or reserved).

```admonish warning "Avoid names longer than 11 letters on 32-Bit"

Rhai uses [`SmartString`] which avoids allocations unless a string is over its internal limit
(23 ASCII characters on 64-bit, but only 11 ASCII characters on 32-bit).

On 64-bit systems, _most_ variable names are shorter than 23 letters, so this is unlikely to become
an issue.

However, on 32-bit systems, take care to limit, where possible, variable names to within 11 letters.
This is particularly true for local variables inside a hot loop, where they are created and
destroyed in rapid succession.

~~~js
// The following is SLOW on 32-bit
for my_super_loop_variable in array {
    print(`Super! ${my_super_loop_variable}`);
}

// Suggested revision:
for loop_var in array {
    print(`Super! ${loop_var}`);
}
~~~
```


Declare a Variable
------------------

Variables are declared using the `let` keyword.

Variables do not have to be given an initial value.
If none is provided, it defaults to [`()`].

A variable defined within a statement block is _local_ to that block.

Use `is_def_var` to detect if a variable is defined.

```rust,no_run
let x;              // ok - value is '()'
let x = 3;          // ok
let _x = 42;        // ok
let x_ = 42;        // also ok
let _x_ = 42;       // still ok

let _ = 123;        // <- syntax error: illegal variable name
let _9 = 9;         // <- syntax error: illegal variable name

let x = 42;         // variable is 'x', lower case
let X = 123;        // variable is 'X', upper case

print(x);           // prints 42
print(X);           // prints 123

{
    let x = 999;    // local variable 'x' shadows the 'x' in parent block

    print(x);       // prints 999
}

print(x);           // prints 42 - the parent block's 'x' is not changed

let x = 0;          // new variable 'x' shadows the old 'x'

print(x);           // prints 0

is_def_var("x") == true;

is_def_var("_x") == true;

is_def_var("y") == false;
```


Use Before Definition
---------------------

By default, variables do not need to be defined before they are used.

If a variable accessed by a script is not defined previously within the same script, it is assumed
to be provided via an external custom [`Scope`] passed to the [`Engine`] via the
`Engine::XXX_with_scope` API.

```rust,no_run
let engine = Engine::new();

engine.run("print(answer)")?;       // <- error: variable 'answer' not found

// Create custom scope
let mut scope = Scope::new();

// Add variable to custom scope
scope.push("answer", 42_i64);

// Run with custom scope
engine.run_with_scope(&mut scope,
    "print(answer)"                 // <- prints 42
)?;
```

If no [`Scope`] is used to evaluate the script (e.g. when using `Engine::run` instead of
`Engine::run_with_scope`), only then will an undefined variable cause a runtime error when accessed.


Strict Variables Mode
---------------------

With [`Engine::set_strict_variables`][options], it is possible to turn on
[_Strict Variables_][strict variables] mode.

When [strict variables] mode is active, accessing a variable not previously defined within
the same script directly causes a parse error when compiling the script.

Turn on [strict variables] mode if no [`Scope`] is to be provided for script evaluation runs.
This way, variable access errors are caught during compile time instead of runtime.

```rust,no_run
let x = 42;

print(x);           // prints 42

print(foo);         // <- parse error under strict variables mode:
                    //    variable 'foo' is undefined
```
