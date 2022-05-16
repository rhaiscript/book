Throw Exception on Error
========================

To deliberately return an error, use the `throw` keyword.

```js
if some_bad_condition_has_happened {
    throw error;    // 'throw' any value as the exception
}

throw;              // defaults to '()'
```


Catch a Thrown Exception
------------------------

It is possible to _catch_ an exception, instead of having it abort the script run, via the
[`try` ... `catch`](try-catch.md) statement common to many C-like languages.

```js
fn code_that_throws() {
    throw 42;
}

try
{
    code_that_throws();
}
catch (err)         // 'err' captures the thrown exception value
{
    print(err);     // prints 42
}
```
