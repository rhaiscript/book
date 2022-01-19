Statements
==========

{{#include ../links.md}}


Statements are terminated by semicolons `;` and they are mandatory,
except for the _last_ statement in a _block_ (enclosed by `{` ... `}` pairs) where it can be omitted.

Semicolons can also be omitted for statement types that always end in a block &ndash; for example
the [`if`], [`while`], [`for`],  [`loop`] and [`switch`] statements.

```rust,no_run
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


Closed Scope
------------

A statement block forms a _closed_ scope. Any [variable] or [constant] defined within the block are
removed outside the block.

```rust,no_run
let x = 42;
let y = 18;

{
    const HELLO = 99;
    let y = 0;

    print(y + HELLO);   // prints 99 (y is zero)
        :    
}                       // <- 'HELLO' and 'y' go away here...

print(x + y);           // prints 60 (y is still 18)

print(HELLO);           // <- error: 'HELLO' not found
```
