Assignments
===========

Value assignments to [variables](variables.md) use the `=` symbol.

```rust
let foo = 42;

bar = 123 * 456 - 789;

x[1][2].prop = do_calculation();
```


Valid Assignment Targets
------------------------

The left-hand-side (LHS) of an assignment statement must be a valid
_[l-value](https://en.wikipedia.org/wiki/Value_(computer_science))_, which must be rooted in a
[variable](variables.md), potentially extended via indexing or properties.

~~~admonish bug "Assigning to invalid l-value"

Expressions that are not valid _l-values_ cannot be assigned to.

```rust
x = 42;                 // variable is an l-value

x[1][2][3] = 42         // variable indexing is an l-value

x.prop1.prop2 = 42;     // variable property is an l-value

foo(x) = 42;            // syntax error: function call is not an l-value

x.foo() = 42;           // syntax error: method call is not an l-value

(x + y) = 42;           // syntax error: binary op is not an l-value
```
~~~

Values are Cloned
-----------------

Values assigned are always _cloned_.
So care must be taken when assigning large data types (such as [arrays](arrays.md)).

```rust
x = y;                  // value of 'y' is cloned

x == y;                 // both 'x' and 'y' hold different copies
                        // of the same value
```


Moving Data
-----------

When assigning large data types, sometimes it is desirable to _move_ the data instead of cloning it.

Use the `take` function to _move_ data.

### The original variable is left with `()`

```rust
x = take(y);            // value of 'y' is moved to 'x'

y == ();                // 'y' now holds '()'

x != y;                 // 'x' holds the original value of 'y'
```

### Return large data types from functions

`take` is convenient when returning large data types from a [function](functions.md).

```rust
fn get_large_value_naive() {
    let large_result = do_complex_calculation();

    large_result.done = true;

    // Return a cloned copy of the result, then the
    // local variable 'large_result' is thrown away!
    large_result
}

fn get_large_value_smart() {
    let large_result = do_complex_calculation();

    large_result.done = true;

    // Return the result without cloning!
    // Method style call is also OK.
    large_result.take()
}
```

### Assigning large data types to object map properties

`take` is useful when assigning large data types to [object map](object-maps.md) properties.

```rust
let x = [];

// Build a large array
for n in 0..1000000 { x += n; }

// The following clones the large array from 'x'.
// Both 'my_object.my_property' and 'x' now hold exact copies
// of the same large array!
my_object.my_property = x;

// Move it to object map property via 'take' without cloning.
// 'x' now holds '()'.
my_object.my_property = x.take();

// Without 'take', the following must be done to avoid cloning:
my_object.my_property = [];

for n in 0..1000000 { my_object.my_property += n; }
```
