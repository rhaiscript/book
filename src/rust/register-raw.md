Use the Low-Level API to Register a Rust Function
================================================

{{#include ../links.md}}

When a native Rust function is registered with an [`Engine`] using the `register_XXX` API, Rhai
transparently converts all function arguments from [`Dynamic`] into the correct types before calling
the function.

For more power and flexibility, there is a _low-level_ API to work directly with [`Dynamic`] values
without the conversions.


Raw Function Registration
-------------------------

The `Engine::register_raw_fn` method is marked _volatile_, meaning that it may be changed without warning.

If this is acceptable, then using this method to register a Rust function opens up more opportunities.

```rust,no_run
engine.register_raw_fn(
    "increment_by",                                         // function name
    &[                                                      // a slice containing parameter types
        std::any::TypeId::of::<i64>(),                      // type of first parameter
        std::any::TypeId::of::<i64>()                       // type of second parameter
    ],
    |context, args| {                                       // fixed function signature
        // Arguments are guaranteed to be correct in number and of the correct types.

        // But remember this is Rust, so you can keep only one mutable reference at any one time!
        // Therefore, get a '&mut' reference to the first argument _last_.
        // Alternatively, use `args.split_first_mut()` etc. to split the slice first.

        let y = *args[1].read_lock::<i64>().unwrap();       // get a reference to the second argument
                                                            // then copy it because it is a primary type

        let y = std::mem::take(args[1]).cast::<i64>();      // alternatively, directly 'consume' it

        let y = args[1].as_int().unwrap();                  // alternatively, use 'as_xxx()'

        let x = args[0].write_lock::<i64>().unwrap();       // get a '&mut' reference to the first argument

        *x += y;                                            // perform the action

        Ok(Dynamic::UNIT)                                   // must be 'Result<Dynamic, Box<EvalAltResult>>'
    }
);

// The above is the same as (in fact, internally they are equivalent):

engine.register_fn("increment_by", |x: &mut i64, y: i64| *x += y);
```


Function Signature
------------------

The function signature passed to `Engine::register_raw_fn` takes the following form.

> `Fn(context: NativeCallContext, args: &mut [&mut Dynamic]) -> Result<T, Box<EvalAltResult>>`

where:

| Parameter |         Type          | Description                                                                                                                                 |
| --------- | :-------------------: | ------------------------------------------------------------------------------------------------------------------------------------------- |
| `T`       |     `impl Clone`      | return type of the function                                                                                                                 |
| `context` | [`NativeCallContext`] | the current _native call context_, useful for recursively calling functions on the same [`Engine`]                                          |
| `args`    | `&mut [&mut Dynamic]` | a slice containing `&mut` references to [`Dynamic`] values.<br/>The slice is guaranteed to contain enough arguments _of the correct types_. |

### Return value

The return value is the result of the function call.

Remember, in Rhai, all arguments _except_ the _first_ one are always passed by _value_ (i.e. cloned).
Therefore, it is unnecessary to ever mutate any argument except the first one, as all mutations
will be on the cloned copy.


Extract The First `&mut` Argument (If Any)
-----------------------------------------

To extract the first `&mut` argument passed by reference from the `args` parameter (`&mut [&mut Dynamic]`),
use the following to get a mutable reference to the underlying value:

```rust,no_run
let value: &mut T = &mut *args[0].write_lock::<T>().unwrap();

*value = ...    // overwrite the existing value of the first `&mut` parameter
```

When there is a mutable reference to the first `&mut` argument, there can be no other immutable
references to `args`, otherwise the Rust borrow checker will complain.

Therefore, always extract the mutable reference last, _after_ all other arguments are taken.


Extract Other Pass-By-Value Arguments
------------------------------------

To extract an argument passed by value from the `args` parameter (`&mut [&mut Dynamic]`), use the following statements.

| Argument type             | Access statement (`n` = argument position)          |                 Result                  | Original value |
| ------------------------- | --------------------------------------------------- | :-------------------------------------: | :------------: |
| `INT`                     | `args[n].as_int().unwrap()`                         |                  `INT`                  |   untouched    |
| `FLOAT`                   | `args[n].as_float().unwrap()`                       |                 `FLOAT`                 |   untouched    |
| [`Decimal`][rust_decimal] | `args[n].as_decimal().unwrap()`                     |        [`Decimal`][rust_decimal]        |   untouched    |
| `bool`                    | `args[n].as_bool().unwrap()`                        |                 `bool`                  |   untouched    |
| `char`                    | `args[n].as_char().unwrap()`                        |                 `char`                  |   untouched    |
| `()`                      | `args[n].as_unit().unwrap()`                        |                  `()`                   |   untouched    |
| [String]                  | `&*args[n].read_lock::<ImmutableString>().unwrap()` | [`&ImmutableString`][`ImmutableString`] |   untouched    |
| [String] (consumed)       | `std::mem::take(args[n]).cast::<ImmutableString>()` |           [`ImmutableString`]           |     [`()`]     |
| Others                    | `&*args[n].read_lock::<T>().unwrap()`               |                  `&T`                   |   untouched    |
| Others (consumed)         | `std::mem::take(args[n]).cast::<T>()`               |                   `T`                   |     [`()`]     |


Example &ndash; Pass a Callback to a Rust Function
-------------------------------------------------

The low-level API is useful when there is a need to interact with the scripting [`Engine`]
within a function.

The following example registers a function that takes a [function pointer] as an argument,
then calls it within the same [`Engine`].  This way, a _callback_ function can be provided
to a native Rust function.

The example also showcases the use of `FnPtr::call_raw`, a low-level API which allows binding the
`this` pointer to the function pointer call.

```rust,no_run
use rhai::{Engine, FnPtr};

let mut engine = Engine::new();

// Register a Rust function
engine.register_raw_fn(
    "bar",
    &[
        std::any::TypeId::of::<i64>(),                      // parameter types
        std::any::TypeId::of::<FnPtr>(),
        std::any::TypeId::of::<i64>(),
    ],
    |context, args| {
        // 'args' is guaranteed to contain enough arguments of the correct types

        let fp = std::mem::take(args[1]).cast::<FnPtr>();   // 2nd argument - function pointer
        let value = std::mem::take(args[2]);                // 3rd argument - function argument

        // The 1st argument holds the 'this' pointer.
        // This should be done last as it gets a mutable reference to 'args'.
        let this_ptr = args.get_mut(0).unwrap();

        // Use 'FnPtr::call_raw' to call the function pointer with the context
        // while also binding the 'this' pointer!
        fp.call_raw(&context, Some(this_ptr), [value])
    },
);

let result = engine.eval::<i64>(
r#"
    fn foo(x) { this += x; }        // script-defined function 'foo'

    let x = 41;                     // object
    x.bar(Fn("foo"), 1);            // pass 'foo' as function pointer
    x
"#)?;
```

~~~admonish tip "Tip: Hold multiple references"

In order to access a value argument that is expensive to clone _while_ holding a mutable reference
to the first argument, use one of the following tactics:

1. If it is a [primary type][standard types] other than [string], use `as_xxx()` as above;

2. Directly _consume_ that argument via `std::mem::take` as above;

3. Use `split_first_mut` to partition the slice:

```rust,no_run
// Partition the slice
let (first, rest) = args.split_first_mut().unwrap();

// Mutable reference to the first parameter, of type '&mut A'
let this_ptr = &mut *first.write_lock::<A>().unwrap();

// Immutable reference to the second value parameter, of type '&B'
// This can be mutable but there is no point because the parameter is passed by value
let value_ref = &*rest[0].read_lock::<B>().unwrap();
```
~~~


TL;DR
-----

~~~admonish question "Why `read_lock` and `write_lock`?"

The [`Dynamic`] API that casts it to a reference to a particular data type  is `read_lock`
(for an immutable reference) and `write_lock` (for a mutable reference).

As the naming shows, something is _locked_ in order to allow this access, and that something
is a _shared value_ created by [capturing][automatic currying] variables from [closures].

Shared values are implemented as `Rc<RefCell<Dynamic>>` (`Arc<RwLock<Dynamic>>` under [`sync`]).

If the value is _not_ a shared value, or if running under [`no_closure`] where there is
no [capturing][automatic currying], this API de-sugars to a simple reference cast.

In other words, there is no locking and reference counting overhead for the vast majority of
non-shared values.

If the value _is_ a shared value, then it is first _locked_ and the returned _lock guard_
allows access to the underlying value in the specified type.
~~~
