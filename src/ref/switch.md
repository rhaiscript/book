Switch Statement
================

The `switch` statement allows matching on [literal](../appendix/literals.md) values.

```js
switch calc_secret_value(x) {
    1 => print("It's one!"),
    2 => {
        print("It's two!");
        print("Again!");
    }
    3 => print("Go!"),
    // _ is the default when no case matches. It must be the last case.
    _ => print(`Oops! Something's wrong: ${x}`)
}
```


Default Case
------------

A _default_ case (i.e. when no other cases match) can be specified with `_`.

```admonish warning.small "Must be last"

The default case must be the _last_ case in the `switch` statement.
```

```js
switch wrong_default {
    1 => 2,
    _ => 9,     // <- syntax error: default case not the last
    2 => 3,
    3 => 4,     // <- ending with extra comma is OK
}
```


Array and Object Map Literals Also Work
---------------------------------------

The `switch` expression can match against any _[literal](../appendix/literals.md)_, including
[array](arrays.md) and [object map](object-maps.md) [literals](../appendix/literals.md).

```js
// Match on arrays
switch [foo, bar, baz] {
    ["hello", 42, true] => ...,
    ["hello", 123, false] => ...,
    ["world", 1, true] => ...,
    _ => ...
}

// Match on object maps
switch map {
    #{ a: 1, b: 2, c: true } => ...,
    #{ a: 42, d: "hello" } => ...,
    _ => ...
}
```


Case Conditions
---------------

Similar to Rust, each case (except the default case at the end) can provide an optional condition
that must evaluate to `true` in order for the case to match.

Unlike Rust, however, case conditions do not allow the case values to duplicate.

```js
let result = switch calc_secret_value(x) {
    1 if some_external_condition(x, y, z) => 100,

    2 if x < foo => 200,
    2 if bar() => 999,      // <- syntax error: still cannot have duplicated cases

    3 => if CONDITION {     // <- put condition inside statement block for
        123                 //    duplicated cases
    } else {
        0
    }

    _ if CONDITION => 8888  // <- syntax error: default case cannot have condition
};
```


Range Cases
-----------

Because of their popularity, [literal](../appendix/literals.md) integer [ranges](ranges.md) can also
be used as `switch` cases.

Numeric [ranges](ranges.md) are only searched when the `switch` value is itself an integer
(i.e. they never match any other data types).

```admonish warning.small "Must come after integer cases"

Range cases must come _after_ all integer cases.
```

```js
let x = 42;

switch x {
    'x' => ...,             // no match: wrong data type

    1 => ...,               // <- specific integer cases are checked first
    2 => ...,               // <- but these do not match

    0..50 if x > 45 => ..., // no match: condition is 'false'

    -10..20 => ...,         // no match: not in range

    0..50 => ...,           // <- MATCH!!! duplicated range cases are OK

    30..100 => ...,         // no match: even though it is within range,
                            // the previous case matches first

    42 => ...,              // <- syntax error: integer cases cannot follow range cases
}
```

```admonish tip.small "Tip: Ranges can overlap"

When more then one [range](ranges.md) contain the `switch` value, the _first_ one with a fulfilled condition
(if any) is evaluated.

Numeric [range](ranges.md) cases are tried in the order that they appear in the original script.
```


Difference From `if`-`else if` Chain
------------------------------------

Although a `switch` expression looks _almost_ the same as an [`if`-`else if`](if.md) chain, there
are subtle differences between the two.

### Look-up Table vs `x == y`

A `switch` expression matches through _hashing_ via a look-up table. Therefore, matching is very
fast.  Walking down an [`if`-`else if`](if.md) chain is _much_ slower.

On the other hand, operators can be [overloaded](overload.md) in Rhai, meaning that it is possible
to override the `==` operator for integers such that `x == y` returns a different result from the
built-in default.

`switch` expressions do _not_ use the `==` operator for comparison; instead, they _hash_ the data
values and jump directly to the correct statements via a pre-compiled look-up table.  This makes
matching extremely efficient, but it also means that [overloading](overload.md) the `==` operator
will have no effect.

Therefore, in environments where it is desirable to [overload](overload.md) the `==` operator for
[standard types](values-and-types.md) &ndash; though it is difficult to think of valid scenarios
where you'd want `1 == 1` to return something other than `true` &ndash; avoid using the `switch`
expression.

### Efficiency

Because the `switch` expression works through a look-up table, it is very efficient even for _large_
number of cases; in fact, switching is an O(1) operation regardless of the size of the data and
number of cases to match.

A long [`if`-`else if`](if.md) chain becomes increasingly slower with each additional case because
essentially an O(n) _linear scan_ is performed.


Switch Expression
=================

Like [`if`](if.md), `switch` also works as an _expression_.

```admonish tip.small "Tip"

This means that a `switch` expression can appear anywhere a regular expression can,
e.g. as [function](functions.md) call arguments.
```

```js
let x = switch foo { 1 => true, _ => false };

func(switch foo {
    "hello" => 42,
    "world" => 123,
    _ => 0
});

// The above is somewhat equivalent to:

let x = if foo == 1 { true } else { false };

if foo == "hello" {
    func(42);
} else if foo == "world" {
    func(123);
} else {
    func(0);
}
```
