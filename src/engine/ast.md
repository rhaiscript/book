Manage `AST`'s
==============

{{#include ../links.md}}


When compiling a Rhai script to an [`AST`], the following data are packaged together as a single unit:

| Data                                                           |                   Type                    | Description                                                                              |                                             Access API                                             |
| -------------------------------------------------------------- | :---------------------------------------: | ---------------------------------------------------------------------------------------- | :------------------------------------------------------------------------------------------------: |
| Source name                                                    |            [`ImmutableString`]            | optional text name to identify the source of the script                                  | `source(&self)`, `clone_source(&self)`, `set_source(&mut self, source)`, `clear_source(&mut self)` |
| Statements                                                     |                `Vec<Stmt>`                | list of script statements at global level                                                |       `statements(&self)`, `statements_mut(&mut self)` (only available under [`internals`])        |
| Functions (not available under [`no_function`])                |                [`Module`]                 | functions defined in the script                                                          |               `shared_lib(&self)`, `lib(&self)` (only available under [`internals`])               |
| Embedded [module resolver] (not available under [`no_module`]) | [`StaticModuleResolver`][module resolver] | embedded [module resolver] for [self-contained `AST`](../rust/modules/self-contained.md) |                       `resolver(&self)` (only available under [`internals`])                       |

Use the source name to identify the source script in errors &ndash; useful when multiple [modules]
are imported recursively.

Most of the [`AST`] API is available only under the [`internals`] feature.

For the complete [`AST`] API, refer to the [documentation](https://docs.rs/rhai/{{version}}/rhai/struct.AST.html) online.


Extract Only Functions
----------------------

The following methods, not available under [`no_function`], allow manipulation of the functions
encapsulated within an [`AST`]:

| Method                                         | Description                                                                                                   |
| ---------------------------------------------- | ------------------------------------------------------------------------------------------------------------- |
| `clone_functions_only(&self)`                  | clone the [`AST`] into a new [`AST`] with only functions, excluding statements                                |
| `clone_functions_only_filtered(&self, filter)` | clone the [`AST`] into a new [`AST`] with only functions that pass the filter predicate, excluding statements |
| `retain_functions(&mut self, filter)`          | remove all functions in the [`AST`] that do not pass a particular predicate filter; statements are untouched  |
| `iter_functions(&self)`                        | return an iterator on all the functions in the [`AST`]                                                        |
| `clear_functions(&mut self)`                   | remove all functions from the [`AST`], leaving only statements                                                |


Extract Only Statements
-----------------------

The following methods allow manipulation of the statements in an [`AST`]:

| Method                         | Description                                                                        |
| ------------------------------ | ---------------------------------------------------------------------------------- |
| `clone_statements_only(&self)` | clone the [`AST`] into a new [`AST`] with only the statements, excluding functions |
| `clear_statements(&mut self)`  | remove all statements from the [`AST`], leaving only functions                     |


Merge and Combine AST's
-----------------------

The following methods merge one [`AST`] with another:

| Method                                        | Description                                                                                                                                                                                                                                                                                   |
| --------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `merge(&self, &second)`, `+` operator         | append the second [`AST`] to this [`AST`], yielding a new [`AST`] that is a combination of the two; statements are simply appended, functions in the second [`AST`] of the same name and arity override similar functions in this [`AST`]                                                     |
| `merge_filtered(&self, &second, filter)`      | append the second [`AST`] (but only functions that pass the predicate filter) to this [`AST`], yielding a new [`AST`] that is a combination of the two; statements are simply appended, functions in the second [`AST`] of the same name and arity override similar functions in this [`AST`] |
| `combine(&mut self, second)`, `+=` operator   | append the second [`AST`] to this [`AST`]; statements are simply appended, functions in the second [`AST`] of the same name and arity override similar functions in this [`AST`]                                                                                                              |
| `combine_filtered(&mut self, second, filter)` | append the second [`AST`] (but only functions that pass the predicate filter) to this [`AST`]; statements are simply appended, functions in the second [`AST`] of the same name and arity override similar functions in this [`AST`]                                                          |

When statements are appended, beware that this may change the semantics of the script.

```rust , no_run
// First script
let ast1 = engine.compile(
"
     fn foo(x) { 42 + x }
     foo(1)
")?;

// Second script
let ast2 = engine.compile(
r#"
     fn foo(n) { `hello${n}` }
     foo("!")
"#)?;

// Merge them
let merged = ast1.merge(&ast2);

// Notice that using the '+' operator also works:
let merged = &ast1 + &ast2;
```

`merged` in the above example essentially contains the following script program:

```js
fn foo(n) { `hello${n}` }   // <- definition of first 'foo' is overwritten
foo(1)                      // <- notice this will be "hello1" instead of 43,
                            //    but it is no longer the return value
foo("!")                    // <- returns "hello!"
```


Walk an AST
-----------

The [`internals`] feature allows access to internal Rhai data structures, particularly the nodes
that make up the [`AST`].

### AST node types

There are a few useful types when walking an [`AST`]:

| Type         | Description                                                       |
| ------------ | ----------------------------------------------------------------- |
| `ASTNode`    | an `enum` with two variants: `Expr` or `Stmt`                     |
| `Expr`       | an _expression_                                                   |
| `Stmt`       | a _statement_                                                     |
| `BinaryExpr` | a sub-type containing the LHS and RHS of a binary expression      |
| `FnCallExpr` | a sub-type containing information on a function call              |
| `CustomExpr` | a sub-type containing information on a [custom syntax] expression |

The `AST::walk` method takes a callback function and recursively walks the [`AST`] in depth-first
manner, with the parent node visited before its children.

### Callback function signature

The function signature of the callback function is:

> `FnMut(&[ASTNode]) -> bool`

The single argument passed to the method contains a slice of `ASTNode` types representing the path
from the current node to the root of the [`AST`].

Return `true` to continue walking the [`AST`], or `false` to terminate.

### Children visit order

The order of visits to the children of each node type:

| Node type                                        | Children visit order                                                                       |
| ------------------------------------------------ | ------------------------------------------------------------------------------------------ |
| `if` statement                                   | condition expression, _then_ statements, _else_ statements (if any)                        |
| `switch` statement                               | match element, each of the case conditions and statements, default statements (if any)     |
| `while`, `do`, `loop` statement                  | condition expression, statements body                                                      |
| `for` statement                                  | collection expression, statements body                                                     |
| `try` ... `catch` statement                      | `try` statements body, `catch` statements body                                             |
| `return` statement                               | return value expression                                                                    |
| [`import`] statement                             | path expression                                                                            |
| [Array] literal                                  | each of the element expressions                                                            |
| [Object map] literal                             | each of the element expressions                                                            |
| Interpolated [string]                            | each of the [string]/expression segments                                                   |
| Indexing                                         | LHS, RHS                                                                                   |
| Field access/method call                         | LHS, RHS                                                                                   |
| `&&`, <code>\|\|</code>                          | LHS, RHS                                                                                   |
| [Function] call, operator expression             | each of the argument expressions                                                           |
| [`let`][variable], [`const`][constant] statement | value expression                                                                           |
| Assignment statement                             | l-value expression, value expression                                                       |
| Statements block                                 | each of the statements                                                                     |
| Custom syntax expression                         | each of the `$expr$`, `$stmt$`, `$ident$`, `$bool$`, `$int$`, `$float$`, `$string$` blocks |
| All others                                       | single child, or none                                                                      |
