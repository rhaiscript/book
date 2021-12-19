`in` Operator
=============

{{#include ../links.md}}


The `in` operator is used to check for _containment_ &ndash; i.e. whether a particular collection
data type _contains_ a particular item.

Internally the `in` operator is simply syntactic sugar for a call to the `contains` function.


Built-in Support
----------------

The following standard data types have built-in support for the `in` operator.

|    Data type    |              Check for              |
| :-------------: | :---------------------------------: |
| Numeric [range] |           integer number            |
|     [Array]     |           contained item            |
|  [Object map]   |            property name            |
|    [String]     | [sub-string][string] or [character] |


Examples
--------

```rust no_run
42 in [1, "abc", 42, ()] == true;   // check array for item

"foo" in #{                         // check object map for property name
    foo: 42,
    bar: true,
    baz: "hello"
} == true;

'w' in "hello, world!" == true;     // check string for character

"wor" in "hello, world" == true;    // check string for sub-string

42 in -100..100 == true;            // check range for number
```


Array Items Comparison
----------------------

The default implementation of the `in` operator for [arrays] uses the `==` operator (if defined)
to compare items.

Beware that, for a [custom type], `==` defaults to `false` when comparing it with a value of of the
same type.

See the section on [_Logic Operators_](logic.md) for more details.

```rust no_run
let ts = new_ts();                  // assume 'new_ts' returns a custom type

let a = [1, 2, 3, ts, 42, 999];     // array contains custom type

42 in a == true;                    // 42 cannot be compared with 'ts'
                                    // so it defaults to 'false'
                                    // because == operator is not defined
```


Custom Implementation of `contains`
----------------------------------

The `in` operator maps directly to a call to a function `contains` with the two operands switched.

For example:

```rust no_run
item in container
```

maps to the following function call:

```rust no_run
contains(container, item)
```

Support for the `in` operator can be easily extended to other types by registering a custom binary
function named `contains` with the correct parameter types.

For example:

```rust no_run
engine.register_type::<TestStruct>()
      .register_fn("new_ts", || TestStruct::new())
      .register_fn("contains", |container: &mut TestStruct, item: i64| -> bool {
          // Remember the parameters are switched from the 'in' expression
          container.contains(item)
      });
```

Now the `in` operator can be used for `TestStruct`:

```rust no_run
let ts = new_ts();

if 42 in ts {                       // this calls the 'contains' function
    print("I got 42!");
}

let err = "hello" in ts;            // <- runtime error: 'contains' not found
```
