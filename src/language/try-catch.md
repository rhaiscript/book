Catch Exceptions
================

{{#include ../links.md}}


When an [exception] is thrown via a [`throw`] statement, evaluation of the script halts
and the [`Engine`] returns with `Err(Box<EvalAltResult::ErrorRuntime>)` containing the
exception value that has been thrown.

It is possible, via the `try` ... `catch` statement, to _catch_ exceptions, optionally
with an _error variable_.

```rust
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


Re-Throw Exception
------------------

Like the `try` ... `catch` syntax in most languages, it is possible to _re-throw_
an exception within the `catch` block simply by another [`throw`] statement without
a value.


```rust
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


Catchable Exceptions
--------------------

Many script-oriented exceptions can be caught via `try` ... `catch`:

| Error type                                    |        Error value         |
| --------------------------------------------- | :------------------------: |
| Runtime error thrown by a [`throw`] statement | value in `throw` statement |
| Other runtime error                           |   error message [string]   |
| Arithmetic error                              |   error message [string]   |
| Variable not found                            |   error message [string]   |
| [Function] not found                          |   error message [string]   |
| [Module] not found                            |   error message [string]   |
| Unbound [`this`]                              |   error message [string]   |
| Data type mismatch                            |   error message [string]   |
| Assignment to a calculated/constant value     |   error message [string]   |
| [Array]/[string] indexing out-of-bounds       |   error message [string]   |
| Indexing with an inappropriate data type      |   error message [string]   |
| Error in a dot expression                     |   error message [string]   |
| `for` statement without a [type iterator]     |   error message [string]   |
| Error in an `in` expression                   |   error message [string]   |
| Data race detected                            |   error message [string]   |


Non-Catchable Exceptions
------------------------

Some exceptions _cannot_ be caught:

* Syntax error during parsing
* System error &ndash; e.g. script file not found
* Script evaluation metrics over [safety limits]({{rootUrl}}/safety/index.md)
* Function calls nesting exceeding [maximum call stack depth]
* Script evaluation manually terminated
