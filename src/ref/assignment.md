Assignments
===========

Value assignments to [variables](variables.md) use the `=` symbol.

```rust
let foo = 42;

bar = 123 * 456 - 789;

x[1][2].prop = do_calculation();
```

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

(x + y) = 42;           // syntax error: binary expression is not an l-value
```
~~~


Compound Assignments
====================

Compound assignments are assignments with a [binary operator][operators] attached.

```rust
number += 8;            // number = number + 8

number -= 7;            // number = number - 7

number *= 6;            // number = number * 6

number /= 5;            // number = number / 5

number %= 4;            // number = number % 4

number **= 3;           // number = number ** 3

number <<= 2;           // number = number << 2

number >>= 1;           // number = number >> 1

number &= 0x00ff;       // number = number & 0x00ff;

number |= 0x00ff;       // number = number | 0x00ff;

number ^= 0x00ff;       // number = number ^ 0x00ff;
```


The Flexible `+=`
-----------------

The the `+` and `+=` operators are often [overloaded](overload.md) to perform build-up
operations for different data types.

### Build strings

```rust
let my_str = "abc";

my_str += "ABC";
my_str += 12345;

my_str == "abcABC12345"
```

### Concatenate arrays

```rust
let my_array = [1, 2, 3];

my_array += [4, 5];

my_array == [1, 2, 3, 4, 5];
```

### Concatenate BLOB's

```rust
let my_blob = blob(3, 0x42);

my_blob += blob(5, 0x89);

my_blob.to_string() == "[4242428989898989]";
```

### Mix two object maps together

```rust
let my_obj = #{ a:1, b:2 };

my_obj += #{ c:3, d:4, e:5 };

my_obj == #{ a:1, b:2, c:3, d:4, e:5 };
```

### Add seconds to timestamps

```rust
let now = timestamp();

now += 42.0;

(now - timestamp()).round() == 42.0;
```
