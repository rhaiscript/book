Statements
==========

{{#include ../links.md}}

Terminated by '`;`'
------------------

Statements are terminated by semicolons '`;`' and they are mandatory,
except for the _last_ statement in a _block_ (enclosed by '`{`' .. '`}`' pairs) where it can be omitted.

Semicolons can also be omitted if the statement ends with a block itself
(e.g. the `if`, `while`, `for` and `loop` statements).

```rust
let a = 42;             // normal assignment statement
let a = foo(42);        // normal function call statement
foo < 42;               // normal expression as statement

let a = { 40 + 2 };     // 'a' is set to the value of the statement block, which is the value of the last statement
//              ^ the last statement does not require a terminating semicolon (although it also works with it)
//                ^ semicolon required here to terminate the assignment statement; it is a syntax error without it

if foo { a = 42 }
//               ^ there is no need to terminate an if-statement with a semicolon

4 * 10 + 2              // a statement which is just one expression - no ending semicolon is OK
                        // because it is the last statement of the whole block
```


Statement Expression
--------------------

A statement can be used anywhere where an expression is expected. These are called, for lack of a more
creative name, "statement expressions."

The _last_ statement of a statement block is _always_ the block's return value when used as a statement,
_regardless_ of whether it is terminated by a semicolon or not. This is different from Rust where,
if the last statement is terminated by a semicolon, the block's return value is taken to be `()`.

If the last statement has no return value (e.g. variable definitions, assignments) then it is assumed to be [`()`].
