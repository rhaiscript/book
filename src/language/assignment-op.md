Compound Assignment Operators
=============================

{{#include ../links.md}}


```rust no_run
let number = 9;

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
----------------

The the `+` and `+=` operators are often [overloaded][function overloading] to perform
build-up operations for different data types.

For example, `+=` is commonly used to build [strings]:

```rust no_run
let my_str = "abc";

my_str += "ABC";
my_str += 12345;

my_str == "abcABC12345"
```

to concatenate [arrays]:

```rust no_run
let my_array = [1, 2, 3];

my_array += [4, 5];

my_array == [1, 2, 3, 4, 5];
```

to concatenate [BLOB's]:

```rust no_run
let my_blob = blob(3, 0x42);

my_blob += blob(5, 0x99);

my_blob.to_string() == "[4242429999999999]";
```

to mix two [object maps] together:

```rust no_run
let my_obj = #{ a:1, b:2 };

my_obj += #{ c:3, d:4, e:5 };

my_obj.len() == 5;
```

to add to [timestamps]:

```rust no_run
let now = timestamp();

now += 42;

now - timestamp() == 42;
```
