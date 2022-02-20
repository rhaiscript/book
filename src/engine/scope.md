`Scope` &ndash; Maintaining State
================================

{{#include ../links.md}}

By default, Rhai treats each [`Engine`] invocation as a fresh one, persisting only the functions
that have been registered but no global state.

This gives each evaluation a clean starting slate.

In order to continue using the same global state from one invocation to the next, such a state
(a `Scope`) must be manually created and passed in.

All `Scope` [variables] and [constants] have values that are [`Dynamic`], meaning they can store
values of any type.

Under [`sync`], however, only types that are `Send + Sync` are supported, and the entire `Scope`
itself will also be `Send + Sync`. This is extremely useful in multi-threaded applications.

```admonish info "Shadowing"

A newly-added [variable] or [constant] _[shadows][shadow]_ previous ones of the same name.

In other words, all versions are kept for [variables] and [constants], but only the latest ones can
be accessed via `get_value<T>`, `get_mut<T>` and `set_value<T>`.

Essentially, a `Scope` is always searched in _reverse order_.
```


`Scope` API
-----------

| Method                                        | Description                                                                                                                          |
| --------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------ |
| `new` _instance method_                       | create a new empty `Scope`                                                                                                           |
| `len`                                         | number of [variables]/[constants] currently within the `Scope`                                                                       |
| `rewind`                                      | _rewind_ (i.e. reset) the `Scope` to a particular number of [variables]/[constants]                                                  |
| `clear`                                       | remove all [variables]/[constants] from the `Scope`, making it empty                                                                 |
| `is_empty`                                    | is the `Scope` empty?                                                                                                                |
| `push`, `push_constant`                       | add a new [variable]/[constant] into the `Scope` with a specified value                                                              |
| `push_dynamic`, `push_constant_dynamic`       | add a new [variable]/[constant] into the `Scope` with a [`Dynamic`] value                                                            |
| `contains`                                    | does the particular [variable] or [constant] exist in the `Scope`?                                                                   |
| `get_value<T>`, `get_mut<T>`                  | get the value of a [variable] within the `Scope`                                                                                     |
| `set_value<T>`                                | set the value of a [variable] within the `Scope`, panics if it is [constant]                                                         |
| `is_constant`                                 | is the particular [variable] in the `Scope` a [constant]?                                                                            |
| `set_or_push<T>`                              | set the value of a [variable] within the `Scope` if it exists and is not [constant]; add a new [variable] into the `Scope` otherwise |
| `iter`, `iter_raw`, `IntoIterator::into_iter` | get an iterator to the [variables]/[constants] within the `Scope`                                                                    |
| `Extend::extend`                              | add [variables]/[constants] to the `Scope`                                                                                           |

```admonish info "See also"

For details on the `Scope` API, refer to the
[documentation](https://docs.rs/rhai/{{version}}/rhai/struct.Scope.html) online.
```

```admonish tip "Tip: The lifetime parameter"

The `Scope` has a _lifetime_ parameter, in the vast majority of cases it can be omitted and
automatically inferred to be `'static`.

Currently, that lifetime parameter is not used.  It is there to maintain backwards compatibility
as well as for possible future expansion when references can also be put into the `Scope`.

The lifetime parameter is not guaranteed to remain unused for future versions.

In order to put a `Scope` into a `struct`, use `Scope<'static>`.
```


Example
-------

In the following example, a `Scope` is created with a few initialized variables, then it is threaded
through multiple evaluations.

```rust,no_run
use rhai::{Engine, Scope, EvalAltResult};

let engine = Engine::new();

// First create the state
let mut scope = Scope::new();

// Then push (i.e. add) some initialized variables into the state.
// Remember the system number types in Rhai are i64 (i32 if 'only_i32')
// and f64 (f32 if 'f32_float').
// Better stick to them or it gets hard working with the script.
scope.push("y", 42_i64)
     .push("z", 999_i64)
     .push_constant("MY_NUMBER", 123_i64)       // constants can also be added
     .set_value("s", "hello, world!");          // 'set_value' adds a new variable when one doesn't exist

// First invocation
engine.run_with_scope(&mut scope, 
"
    let x = 4 + 5 - y + z + MY_NUMBER + s.len;
    y = 1;
")?;

// Second invocation using the same state.
// Notice that the new variable 'x', defined previously, is still here.
let result = engine.eval_with_scope::<i64>(&mut scope, "x + y")?;

println!("result: {}", result);                 // prints 1103

// Variable y is changed in the script - read it with 'get_value'
assert_eq!(scope.get_value::<i64>("y").expect("variable y should exist"), 1);

// We can modify scope variables directly with 'set_value'
scope.set_value("y", 42_i64);
assert_eq!(scope.get_value::<i64>("y").expect("variable y should exist"), 42);
```
