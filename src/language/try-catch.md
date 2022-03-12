Catch Exceptions
================

{{#include ../links.md}}


When an [exception] is thrown via a [`throw`] statement, evaluation of the script halts and the
[`Engine`] returns with `EvalAltResult::ErrorRuntime` containing the exception value thrown.

It is possible, via the `try` ... `catch` statement, to _catch_ exceptions, optionally with an
_error variable_.

```js
// Catch an exception and capturing its value
try
{
    throw 42;
}
catch (err)         // 'err' captures the thrown exception value
{
    print(err);     // prints 42
}

// Catch an exception without capturing its value
try
{
    print(42/0);    // deliberate divide-by-zero exception
}
catch               // no error variable - exception value is discarded
{
    print("Ouch!");
}

// Exception in the 'catch' block
try
{
    print(42/0);    // throw divide-by-zero exception
}
catch
{
    print("You seem to be dividing by zero here...");

    throw "die";    // a 'throw' statement inside a 'catch' block
                    // throws a new exception
}
```


~~~admonish tip "Tip: Re-throw exception"

Like the `try` ... `catch` syntax in most languages, it is possible to _re-throw_ an exception
within the `catch` block simply by another [`throw`] statement without a value.


```js
try
{
    // Call something that will throw an exception...
    do_something_bad_that_throws();
}
catch
{
    print("Oooh! You've done something real bad!");

    throw;          // 'throw' without a value within a 'catch' block
                    // re-throws the original exception
}

```
~~~

```admonish success "Catchable exceptions"

Many script-oriented exceptions can be caught via `try` ... `catch`.

| Error type                                            |         Error value          |
| ----------------------------------------------------- | :--------------------------: |
| Runtime error thrown by a [`throw`] statement         | value in [`throw`] statement |
| Arithmetic error                                      |         [object map]         |
| [Variable] not found                                  |         [object map]         |
| [Function] not found                                  |         [object map]         |
| [Module] not found                                    |         [object map]         |
| Unbound `this`                                        |         [object map]         |
| Data type mismatch                                    |         [object map]         |
| Assignment to a calculated/[constant] value           |         [object map]         |
| [Array]/[string]/[bit-field] indexing out-of-bounds   |         [object map]         |
| Indexing with an inappropriate data type              |         [object map]         |
| Error in property access                              |         [object map]         |
| [`for`] statement on a type without a [type iterator] |         [object map]         |
| Data race detected                                    |         [object map]         |
| Other runtime error                                   |         [object map]         |

The error value in the `catch` clause is an [object map] containing information on the particular error,
including its type, line and character position (if any), and source etc.

When the [`no_object`] feature is turned on, however, the error value is a simple [string] description.
```

```admonish failure "Non-catchable exceptions"

Some exceptions _cannot_ be caught.

| Error type                                                              | Notes                             |
| ----------------------------------------------------------------------- | --------------------------------- |
| System error &ndash; e.g. script file not found                         | system errors are not recoverable |
| Syntax error during parsing                                             | invalid script                    |
| [Custom syntax] mismatch error                                          | incompatible [`Engine`] instance  |
| Script evaluation metrics exceeding [limits][safety]                    | [safety] protection               |
| Script evaluation manually [terminated]({{rootUrl}}/safety/progress.md) | [safety] protection               |
```
