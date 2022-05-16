Catch Exceptions
================

When an [exception](throw.md) is thrown via a [`throw`](throw.md) statement, the script halts with
the exception value.

It is possible, via the `try` ... `catch` statement, to _catch_ exceptions, optionally with an
_error variable_.

> `try` `{` ... `}` `catch` `{` ... `}`
>
> `try` `{` ... `}` `catch` `(` _error variable_ `)` `{` ... `}`

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
within the `catch` block simply by another [`throw`](throw.md) statement without a value.

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

| Error type                                                                                      |              Error value               |
| ----------------------------------------------------------------------------------------------- | :------------------------------------: |
| Runtime error thrown by a [`throw`](throw.md) statement                                         | value in [`throw`](throw.md) statement |
| Arithmetic error                                                                                |      [object map](object-maps.md)      |
| [Variable](variables.md) not found                                                              |      [object map](object-maps.md)      |
| [Function](functions.md) not found                                                              |      [object map](object-maps.md)      |
| [Module](modules/index.md) not found                                                            |      [object map](object-maps.md)      |
| Unbound `this`                                                                                  |      [object map](object-maps.md)      |
| Data type mismatch                                                                              |      [object map](object-maps.md)      |
| Assignment to a calculated/[constant](constants.md) value                                       |      [object map](object-maps.md)      |
| [Array](arrays.md)/[string](strings-chars.md)/[bit-field](bit-fields.md) indexing out-of-bounds |      [object map](object-maps.md)      |
| Indexing with an inappropriate data type                                                        |      [object map](object-maps.md)      |
| Error in property access                                                                        |      [object map](object-maps.md)      |
| [`for`](for.md) statement on a type that is not iterable                                        |      [object map](object-maps.md)      |
| Data race detected                                                                              |      [object map](object-maps.md)      |
| Other runtime error                                                                             |      [object map](object-maps.md)      |

The error value in the `catch` clause is an [object map](object-maps.md) containing information on
the particular error, including its type, line and character position (if any), and source etc.
```

```admonish failure "Non-catchable exceptions"

Some system exceptions _cannot_ be caught.

| Error type                                                              | Notes                             |
| ----------------------------------------------------------------------- | --------------------------------- |
| System error &ndash; e.g. script file not found                         | system errors are not recoverable |
| Syntax error during parsing                                             | invalid script                    |
| [Custom syntax] mismatch error                                          | incompatible [`Engine`] instance  |
| Script evaluation metrics exceeding [limits][safety]                    | [safety] protection               |
| Script evaluation manually [terminated]({{rootUrl}}/safety/progress.md) | [safety] protection               |
```
