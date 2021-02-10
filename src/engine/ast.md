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

```rust
// First script
let ast1 = engine.compile(r#"
                fn foo(x) { 42 + x }
                foo(1)
           "#)?;

// Second script
let ast2 = engine.compile(r#"
                fn foo(n) { "hello" + n }
                foo("!")
           "#)?;

// Merge them
let merged = ast1.merge(&ast2);

// Notice that using the '+' operator also works:
let merged = &ast1 + &ast2;
```

`merged` in the above example essentially contains the following script program:

```rust
fn foo(n) { "hello" + n }   // <- definition of first 'foo' is overwritten
foo(1)                      // <- notice this will be "hello1" instead of 43,
                            //    but it is no longer the return value
foo("!")                    // <- returns "hello!"
```
