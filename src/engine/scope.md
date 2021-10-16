`Scope` &ndash; Initializing and Maintaining State
=================================================

{{#include ../links.md}}

By default, Rhai treats each [`Engine`] invocation as a fresh one, persisting only the functions
that have been defined but no global state.

This gives each evaluation a clean starting slate.

In order to continue using the same global state from one invocation to the next, such a state
(a `Scope`) must be manually created and passed in.

All `Scope` [variables] and [constants] have values that are [`Dynamic`], meaning they can store
values of any type.

Under [`sync`], however, only types that are `Send + Sync` are supported, and the entire `Scope`
itself will also be `Send + Sync`. This is extremely useful in multi-threaded applications.


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
| `get_value<T>`, `get_mut<T>`, `set_value<T>`  | get/set the value of a [variable] within the `Scope` (panics if trying to set the value of a [constant])                             |
| `is_constant`                                 | is the particular [variable] in the `Scope` a [constant]?                                                                            |
| `set_or_push<T>`                              | set the value of a [variable] within the `Scope` if it exists and is not [constant]; add a new [variable] into the `Scope` otherwise |
| `iter`, `iter_raw`, `IntoIterator::into_iter` | get an iterator to the [variables]/[constants] within the `Scope`                                                                    |
| `Extend::extend`                              | add [variables]/[constants] to the `Scope`                                                                                           |

For details on the `Scope` API, refer to the
[documentation](https://docs.rs/rhai/{{version}}/rhai/struct.Scope.html) online.


Shadowing
---------

A newly-added [variable] or [constant] _shadows_ previous ones of the same name.

In other words, all versions are kept for [variables] and [constants], but only the latest ones can
be accessed via `get_value<T>`, `get_mut<T>` and `set_value<T>`.

Essentially, a `Scope` is always searched in _reverse order_.


Example
-------

In the following example, a `Scope` is created with a few initialized variables, then it is threaded
through multiple evaluations.

```rust no_run
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
     .set_value("s", "hello, world!");          // 'set_value' adds a variable when one doesn't exist

// First invocation
engine.eval_with_scope::<()>(&mut scope, 
"
    let x = 4 + 5 - y + z + MY_NUMBER + s.len;
    y = 1;
")?;

// Second invocation using the same state
let result = engine.eval_with_scope::<i64>(&mut scope, "x")?;

println!("result: {}", result);                 // prints 1102

// Variable y is changed in the script - read it with 'get_value'
assert_eq!(scope.get_value::<i64>("y").expect("variable y should exist"), 1);

// We can modify scope variables directly with 'set_value'
scope.set_value("y", 42_i64);
assert_eq!(scope.get_value::<i64>("y").expect("variable y should exist"), 42);
```


`Scope` Lifetime &ndash; Avoid Allocations for Variable Names
-----------------------------------------------------------

The `Scope` has a _lifetime_ parameter, in the vast majority of cases it can be omitted and
automatically inferred to be `'static`.

The reason for such a lifetime parameter is obviously due to something held inside the `Scope`
itself being a reference with a lifetime, and that "something" is the name of each [variable] (and
[constant]) stored within the `Scope`.

Names of [variables] and [constants] are strings, but they do not need to be owned `String` types.

In fact, the names can easily be string slices referencing external data.  This way, no additional
`String` allocations are needed in order to push a [variable] or [constant] into the `Scope`.

For applications where [variables] and/or [constants] are frequently pushed into and removed from
a `Scope` in order to run custom scripts, this has significant performance implications.

```rust no_run
let mut scope = Scope::new();

scope.push("my_var", 42_i64);                   // &'static str

scope.push(String::from("also_var"),            // String
    123_i64
);

// Read a bunch of configuration values from a database
let items: Vec<_> = script_env.iter()
                              .map(|id| read_from_db(id))
                              .collect();

for item in items {
    // No String allocation for variable name
    // 'scope' now has lifetime of 'items'
    scope.push(&item.name, item.value);         // borrowed &str
}
```
