Statements
==========

Statements are terminated by semicolons `;` and they are mandatory, except for the _last_ statement
in a _block_ (enclosed by `{` ... `}` pairs) where it can be omitted.

Semicolons can also be omitted for statement types that always end in a block &ndash; for example
the [`if`](if.md), [`while`](while.md), [`for`](for.md),  [`loop`](loop.md) and
[`switch`](switch.md) statements.

```rust
let a = 42;             // normal assignment statement
let a = foo(42);        // normal function call statement
foo < 42;               // normal expression as statement

let a = { 40 + 2 };     // 'a' is set to the value of the statement block, which is the value of the last statement
//              ^ the last statement does not require a terminating semicolon (but also works with it)
//                ^ semicolon required here to terminate the 'let' statement
//                  it is a syntax error without it, even though it ends with '}'
//                  that is because the 'let' statement doesn't end in a block

if foo { a = 42 }
//               ^ no need to terminate an if-statement with a semicolon
//                 that is because the 'if' statement ends in a block

4 * 10 + 2              // a statement which is just one expression - no ending semicolon is OK
                        // because it is the last statement of the whole block
```


Statement Block
---------------

### Syntax

Statement blocks in Rhai are formed by enclosing zero or more statements within braces `{`...`}`.

> `{` _statement_`;` _statement_`;` ... _statement_ `}`
>
> `{` _statement_`;` _statement_`;` ... _statement_`;` `}`      `// trailing semi-colon is optional`

### Closed scope

A statement block forms a _closed_ scope.

Any [variable](variable.md) and/or [constant](constant.md) defined within the block are removed
outside the block, so are [modules](modules/index.md) [imported](modules/import.md) within the block.

```rust
let x = 42;
let y = 18;

{
    import "hello" as h;
    const HELLO = 99;
    let y = 0;

    h::greet();         // ok

    print(y + HELLO);   // prints 99 (y is zero)

        :    
        :    
}                       // <- 'HELLO' and 'y' go away here...

print(x + y);           // prints 60 (y is still 18)

print(HELLO);           // <- error: 'HELLO' not found

h::greet();             // <- error: module 'h' not found
```


Statement Expression
====================

A statement can be used anywhere where an _expression_ is expected.

These are called, for lack of a more creative name, "statement expressions."

The _last_ statement of a statement block is _always_ the block's return value when used as a statement,
_regardless_ of whether it is terminated by a semicolon or not.

If the last statement has no return value (e.g. variable definitions, assignments) then it is
assumed to be `()`.

```rust
let x = {
    let foo = calc_something();
    let bar = foo + baz;
    bar.further_processing();       // <- this is the return value
};                                  // <- semicolon is needed here...

// The above is equivalent to:
let result;
{
    let foo = calc_something();
    let bar = foo + baz;
    result = bar.further_processing();
}
let x = result;

// Statement expressions can be inserted inside normal expressions
// to avoid duplicated calculations
let x = foo(bar) + { let v = calc(); process(v, v.len, v.abs) } + baz;

// The above is equivalent to:
let foo_result = foo(bar);
let calc_result;
{
    let v = calc();
    result = process(v, v.len, v.abs);  // <- avoid calculating 'v'
}
let x = foo_result + calc_result + baz;

// Statement expressions are also useful as function call arguments
// when side effects are desired
do_work(x, y, { let z = foo(x, y); print(z); z });
           // ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
           //       statement expression
```
