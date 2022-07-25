Manage `AST`'s
==============

{{#include ../links.md}}


When compiling a Rhai script to an [`AST`], the following data are packaged together as a single unit:

| Data                             |                   Type                    | Description                                                                                       |            Requires feature            |                                                   Access API                                                   |
| -------------------------------- | :---------------------------------------: | ------------------------------------------------------------------------------------------------- | :------------------------------------: | :------------------------------------------------------------------------------------------------------------: |
| Source name                      |            [`ImmutableString`]            | optional text name to identify the source of the script                                           |                                        | `source(&self)`,<br/>`clone_source(&self)`,<br/>`set_source(&mut self, source)`,<br/>`clear_source(&mut self)` |
| [Module documentation][comments] |    [`Vec<SmartString>`][`SmartString`]    | documentation of the script                                                                       |              [`metadata`]              |                                    `doc(&self)`,<br/>`clear_doc(&mut self)`                                    |
| Statements                       |                `Vec<Stmt>`                | list of script statements at global level                                                         |             [`internals`]              |                              `statements(&self)`,<br/>`statements_mut(&mut self)`                              |
| Functions                        |       [`Shared<Module>`][`Module`]        | [functions] defined in the script                                                                 | [`internals`],<br/>not [`no_function`] |                                              `shared_lib(&self)`                                               |
| Embedded [module resolver]       | [`StaticModuleResolver`][module resolver] | embedded [module resolver] for [self-contained `AST`]({{rootUrl}}/rust/modules/self-contained.md) |  [`internals`],<br/>not [`no_module`]  |                                               `resolver(&self)`                                                |

Most of the [`AST`] API is available only under the [`internals`] feature.

```admonish tip.small "Tip: Source name"

Use the source name to identify the source script in errors &ndash; useful when multiple [modules]
are imported recursively.
```

~~~admonish info.small "`AST` public API"

For the complete [`AST`] API, refer to the [documentation](https://docs.rs/rhai/{{version}}/rhai/struct.AST.html) online.
~~~


Extract Only Functions
----------------------

The following methods, not available under [`no_function`], allow manipulation of the [functions]
encapsulated within an [`AST`]:

| Method                                         | Description                                                                                                     |
| ---------------------------------------------- | --------------------------------------------------------------------------------------------------------------- |
| `clone_functions_only(&self)`                  | clone the [`AST`] into a new [`AST`] with only [functions], excluding statements                                |
| `clone_functions_only_filtered(&self, filter)` | clone the [`AST`] into a new [`AST`] with only [functions] that pass the filter predicate, excluding statements |
| `retain_functions(&mut self, filter)`          | remove all [functions] in the [`AST`] that do not pass a particular predicate filter; statements are untouched  |
| `iter_functions(&self)`                        | return an iterator on all the [functions] in the [`AST`]                                                        |
| `clear_functions(&mut self)`                   | remove all [functions] from the [`AST`], leaving only statements                                                |


Extract Only Statements
-----------------------

The following methods allow manipulation of the statements in an [`AST`]:

| Method                                                | Description                                                                                     |
| ----------------------------------------------------- | ----------------------------------------------------------------------------------------------- |
| `clone_statements_only(&self)`                        | clone the [`AST`] into a new [`AST`] with only the statements, excluding [functions]            |
| `clear_statements(&mut self)`                         | remove all statements from the [`AST`], leaving only [functions]                                |
| `iter_literal_variables(&self, constants, variables)` | return an iterator on all top-level literal constant and/or variable definitions in the [`AST`] |


Merge and Combine AST's
-----------------------

The following methods merge one [`AST`] with another:

| Method                                        | Description                                                                                                                                                                                                                                                                                         |
| --------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `merge(&self, &ast)`,<br />`+` operator       | append the second [`AST`] to this [`AST`], yielding a new [`AST`] that is a combination of the two; statements are simply appended, [functions] in the second [`AST`] of the same name and arity override similar [functions] in this [`AST`]                                                       |
| `merge_filtered(&self, &ast, filter)`         | append the second [`AST`] (but only [functions] that pass the predicate filter) to this [`AST`], yielding a new [`AST`] that is a combination of the two; statements are simply appended, [functions] in the second [`AST`] of the same name and arity override similar [functions] in this [`AST`] |
| `combine(&mut self, ast)`,<br />`+=` operator | append the second [`AST`] to this [`AST`]; statements are simply appended, [functions] in the second [`AST`] of the same name and arity override similar [functions] in this [`AST`]                                                                                                                |
| `combine_filtered(&mut self, ast, filter)`    | append the second [`AST`] (but only [functions] that pass the predicate filter) to this [`AST`]; statements are simply appended, [functions] in the second [`AST`] of the same name and arity override similar [functions] in this [`AST`]                                                          |

When statements are appended, beware that this may change the semantics of the script.

```rust
// First script
let ast1 = engine.compile(
"
     fn foo(x) { 42 + x }
     foo(1)
")?;

// Second script
let ast2 = engine.compile(
"
     fn foo(n) { `hello${n}` }
     foo("!")
")?;

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

The signature of the callback function takes the following form.

> `FnMut(&[ASTNode]) -> bool`

The single argument passed to the method contains a slice of `ASTNode` types representing the path
from the current node to the root of the [`AST`].

Return `true` to continue walking the [`AST`], or `false` to terminate.

### Children visit order

The order of visits to the children of each node type:

| Node type                                        | Children visit order                                                                                                                                                                           |
| ------------------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [`if`] statement                                 | <ol><li>condition expression</li><li>_then_ statements</li><li>_else_ statements (if any)</li></ol>                                                                                            |
| [`switch`] statement                             | <ol><li>match element</li><li>each of the case conditions and statements, in order</li><li>each of the range conditions and statements, in order</li><li>default statements (if any)</li></ol> |
| [`while`], [`do`], [`loop`] statement            | <ol><li>condition expression</li><li>statements body</li></ol>                                                                                                                                 |
| [`for`] statement                                | <ol><li>collection expression</li><li>statements body</li></ol>                                                                                                                                |
| [`return`] statement                             | return value expression                                                                                                                                                                        |
| [`throw`] statement                              | exception value expression                                                                                                                                                                     |
| [`try` ... `catch`][exception] statement         | <ol><li>`try` statements body</li><li>`catch` statements body</li></ol>                                                                                                                        |
| [`import`] statement                             | path expression                                                                                                                                                                                |
| [Array] literal                                  | each of the element expressions, in order                                                                                                                                                      |
| [Object map] literal                             | each of the element expressions, in order                                                                                                                                                      |
| Interpolated [string]                            | each of the [string]/expression segments, in order                                                                                                                                             |
| Indexing                                         | <ol><li>LHS expression</li><li>RHS (index) expression</li></ol>                                                                                                                                |
| Field access/method call                         | <ol><li>LHS expression</li><li>RHS expression</li></ol>                                                                                                                                        |
| `&&`, <code>\|\|</code>, `??`                    | <ol><li>LHS expression</li><li>RHS expression</li></ol>                                                                                                                                        |
| [Function] call, [operator] expression           | each of the argument expressions, in order                                                                                                                                                     |
| [`let`][variable], [`const`][constant] statement | value expression                                                                                                                                                                               |
| Assignment statement                             | <ol><li>l-value expression</li><li>value expression</li></ol>                                                                                                                                  |
| Statements block                                 | each of the statements, in order                                                                                                                                                               |
| Custom syntax expression                         | each of the inputs stream, in order                                                                                                                                                            |
| All others                                       | single child (if any)                                                                                                                                                                          |
