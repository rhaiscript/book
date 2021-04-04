`switch` Expression
===================

{{#include ../links.md}}

The `switch` _expression_ allows matching on literal values, and it mostly follows Rust's
`match` syntax:

```js,no_run
switch calc_secret_value(x) {
    1 => print("It's one!"),
    2 => {
        print("It's two!");
        print("Again!");
    }
    3 => print("Go!"),
    // _ is the default when no cases match
    _ => print(`Oops! Something's wrong: ${x}`)
}
```


Expression, Not Statement
------------------------

`switch` is not a statement, but an expression. This means that a `switch` expression can
appear anywhere a regular expression can, e.g. as function call arguments.

```ts,no_run
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


Array and Object Map Literals Also Work
--------------------------------------

The `switch` expression can match against any _literal_, including [array] and [object map] literals.

```ts,no_run
// Match on arrays
switch [foo, bar, baz] {
    ["hello", 42, true] => { ... }
    ["hello", 123, false] => { ... }
    ["world", 1, true] => { ... }
    _ => { ... }
}

// Match on object maps
switch map {
    #{ a: 1, b: 2, c: true } => { ... }
    #{ a: 42, d: "hello" } => { ... }
    _ => { ... }
}
```

Switching on [arrays] is very useful when working with Rust enums (see [this chapter]({{rootUrl}}/patterns/enums.md)
for more details).


Difference From `if`-`else if` Chain
-----------------------------------

Although a `switch` expression looks _almost_ the same as an `if`-`else if` chain,
there are subtle differences between the two.

### Look-up Table vs `x == y`

A `switch` expression matches through _hashing_ via a look-up table.
Therefore, matching is very fast.  Walking down an `if`-`else if` chain
is _much_ slower.

On the other hand, operators can be [overloaded][operator overloading] in Rhai,
meaning that it is possible to override the `==` operator for integers such
that `x == y` returns a different result from the built-in default.

`switch` expressions do _not_ use the `==` operator for comparison;
instead, they _hash_ the data values and jump directly to the correct
statements via a pre-compiled look-up table.  This makes matching extremely
efficient, but it also means that [overloading][operator overloading]
the `==` operator will have no effect.

Therefore, in environments where it is desirable to [overload][operator overloading]
the `==` operator for [standard types] &ndash; though it is difficult to think of valid scenarios
where you'd want `1 == 1` to return something other than `true` &ndash;
avoid using the `switch` expression.

### Efficiency

Because the `switch` expression works through a look-up table, it is very efficient
even for _large_ number of cases; in fact, switching is an O(1) operation regardless
of the size of the data and number of cases to match.

A long `if`-`else if` chain becomes increasingly slower with each additional case
because essentially an O(n) _linear scan_ is performed.
