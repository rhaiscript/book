Variables
=========


Valid Names
-----------

Variables in Rhai follow normal C naming rules &ndash; must contain only ASCII letters, digits and underscores `_`.

| Character set | Description              |
| :-----------: | ------------------------ |
|  `A` ... `Z`  | Upper-case ASCII letters |
|  `a` ... `z`  | Lower-case ASCII letters |
|  `0` ... `9`  | Digit characters         |
|      `_`      | Underscore character     |

However, a variable name must also contain at least one ASCII letter, and an ASCII
letter must come _before_ any digits. In other words, the first character that is not an underscore `_`
must be an ASCII letter and not a digit.

```admonish question.side.wide "Why this restriction?"

To reduce confusion (and subtle bugs) because, for instance, `_1` can easily be misread (or mistyped)
as `-1`.

Rhai is dynamic without type checking, so there is no compiler to catch these typos.
```

Therefore, some names, e.g. `_`, `_42foo`, `_1` etc., are not valid in Rhai.

For example: `c3po` and `_r2d2_` are valid variable names, but `3abc` and `____49steps` are not.

Variable names are case _sensitive_.

Variable names also cannot be the same as a [keyword](keywords.md) (active or reserved).

```admonish warning "Avoid names longer than 11 letters on 32-Bit"

Rhai _inlines_ a string, which avoids allocations unless it is over its internal limit
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

```admonish tip.small "Tip: No initial value"

Variables do not have to be given an initial value.
If none is provided, it defaults to `()`.
```

```admonish warning.small "Variables are local"

A variable defined within a [statement block](statements.md) is _local_ to that block.
```

~~~admonish tip.small "Tip: `is_def_var`"

Use `is_def_var` to detect if a variable is defined.
~~~

```rust
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


Shadowing
---------

New variables automatically _shadow_ existing ones of the same name.  There is no error.

```rust
let x = 42;
let y = 123;

print(x);           // prints 42

let x = 88;         // <- 'x' is shadowed here

// At this point, it is no longer possible to access the
// original 'x' on the first line...

print(x);           // prints 88

let x = 0;          // <- 'x' is shadowed again

// At this point, it is no longer possible to access both
// previously-defined 'x'...

print(x);           // prints 0

{
    let x = 999;    // <- 'x' is shadowed in a block
    
    print(x);       // prints 999
}

print(x);           // prints 0 - shadowing within the block goes away

print(y);           // prints 123 - 'y' is not shadowed
```
