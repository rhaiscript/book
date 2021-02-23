Arrays
======

{{#include ../links.md}}

Arrays are first-class citizens in Rhai. Like C, arrays are accessed with zero-based, non-negative integer indices:

> _array_ `[` _index_ `]`

Array literals are built within square brackets `[` ... `]` and separated by commas `,`:

> `[` _value_ `,` _value_ `,` `...` `,` _value_ `]`
>
> `[` _value_ `,` _value_ `,` `...` `,` _value_ `,` `]`     `// trailing comma is OK`

All elements stored in an array are [`Dynamic`], and the array can freely grow or shrink with elements added or removed.

The Rust type of a Rhai array is `rhai::Array`.

[`type_of()`] an array returns `"array"`.

Arrays are disabled via the [`no_index`] feature.

The maximum allowed size of an array can be controlled via `Engine::set_max_array_size`
(see [maximum size of arrays].


Built-in Functions
-----------------

The following methods (mostly defined in the [`BasicArrayPackage`][packages] but excluded if using a [raw `Engine`]) operate on arrays:

| Function                  | Parameter(s)                                                                                                                                                                 | Description                                                                                                                                                                                                               |
| ------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `push`                    | element to insert                                                                                                                                                            | inserts an element at the end                                                                                                                                                                                             |
| `append`                  | array to append                                                                                                                                                              | concatenates the second array to the end of the first                                                                                                                                                                     |
| `+=` operator             | 1) array<br/>2) element to insert (not another array)                                                                                                                        | inserts an element at the end                                                                                                                                                                                             |
| `+=` operator             | 1) array<br/>2) array to append                                                                                                                                              | concatenates the second array to the end of the first                                                                                                                                                                     |
| `+` operator              | 1) first array<br/>2) second array                                                                                                                                           | concatenates the first array with the second                                                                                                                                                                              |
| `==` operator             | 1) first array<br/>2) second array                                                                                                                                           | are the two arrays the same (elements compared with the `==` operator, if defined)?                                                                                                                                       |
| `!=` operator             | 1) first array<br/>2) second array                                                                                                                                           | are the two arrays different (elements compared with the `==` operator, if defined)?                                                                                                                                      |
| `in` operator             | item to find                                                                                                                                                                 | does the array contain the item (compared with the `==` operator, if defined)?                                                                                                                                            |
| `insert`                  | 1) position, beginning if < 0, end if ≥ length<br/>2) element to insert                                                                                                      | inserts an element at a certain index                                                                                                                                                                                     |
| `pop`                     | _none_                                                                                                                                                                       | removes the last element and returns it ([`()`] if empty)                                                                                                                                                                 |
| `shift`                   | _none_                                                                                                                                                                       | removes the first element and returns it ([`()`] if empty)                                                                                                                                                                |
| `extract`                 | 1) start position, beginning if < 0, end if ≥ length<br/>2) _(optional)_ number of items to extract, none if < 0                                                             | extracts a portion of the array into a new array                                                                                                                                                                          |
| `remove`                  | index                                                                                                                                                                        | removes an element at a particular index and returns it ([`()`] if the index is not valid)                                                                                                                                |
| `reverse`                 | _none_                                                                                                                                                                       | reverses the array                                                                                                                                                                                                        |
| `len` method and property | _none_                                                                                                                                                                       | returns the number of elements                                                                                                                                                                                            |
| `pad`                     | 1) target length<br/>2) element to pad                                                                                                                                       | pads the array with an element to at least a specified length                                                                                                                                                             |
| `clear`                   | _none_                                                                                                                                                                       | empties the array                                                                                                                                                                                                         |
| `truncate`                | target length                                                                                                                                                                | cuts off the array at exactly a specified length (discarding all subsequent elements)                                                                                                                                     |
| `chop`                    | target length                                                                                                                                                                | cuts off the head of the array, leaving the tail at exactly a specified length                                                                                                                                            |
| `split`                   | 1) array<br/>2) position to split at, beginning if < 0, end if ≥ length                                                                                                      | splits the array into two arrays, starting from a specified position                                                                                                                                                      |
| `drain`                   | 1) [function pointer] to predicate (usually a [closure])<br/>2) _(optional)_ [function pointer] to function (usually a [closure]) that provides the initial value            | removes all items (returning them) that return `true` when called with the predicate function:<br/>1st parameter: array item<br/>2nd parameter: _(optional)_ offset index                                                 |
| `drain`                   | 1) start position, beginning if < 0, end if ≥ length<br/>2) number of items to remove, none if < 0                                                                           | removes a portion of the array, returning the removed items (not in original order)                                                                                                                                       |
| `retain`                  | 1) [function pointer] to predicate (usually a [closure])<br/>2) _(optional)_ [function pointer] to function (usually a [closure]) that provides the initial value            | removes all items (returning them) that do not return `true` when called with the predicate function:<br/>1st parameter: array item<br/>2nd parameter: _(optional)_ offset index                                          |
| `retain`                  | 1) start position, beginning if < 0, end if ≥ length<br/>2) number of items to retain, none if < 0                                                                           | retains a portion of the array, removes all other items and returning them (not in original order)                                                                                                                        |
| `splice`                  | 1) start position, beginning if < 0, end if ≥ length<br/>2) number of items to remove, none if < 0<br/>3) array to insert                                                    | replaces a portion of the array with another (not necessarily of the same length as the replaced portion)                                                                                                                 |
| `filter`                  | [function pointer] to predicate (usually a [closure])                                                                                                                        | constructs a new array with all items that return `true` when called with the predicate function:<br/>1st parameter: array item<br/>2nd parameter: _(optional)_ offset index                                              |
| `index_of`                | [function pointer] to predicate (usually a [closure])                                                                                                                        | returns the index of the first item in the array that returns `true` when called with the predicate function, or -1 if not found:<br/>1st parameter: array item<br/>2nd parameter: _(optional)_ offset index              |
| `map`                     | [function pointer] to conversion function (usually a [closure])                                                                                                              | constructs a new array with all items mapped to the result of applying the conversion function:<br/>1st parameter: array item<br/>2nd parameter: _(optional)_ offset index                                                |
| `reduce`                  | 1) [function pointer] to accumulator function (usually a [closure])<br/>2) _(optional)_ [function pointer] to function (usually a [closure]) that provides the initial value | reduces the array into a single value via the accumulator function:<br/>1st parameter: accumulated value ([`()`] initially)<br/>2nd parameter: array item<br/>3rd parameter: _(optional)_ offset index                    |
| `reduce_rev`              | 1) [function pointer] to accumulator function (usually a [closure])<br/>2) _(optional)_ [function pointer] to function (usually a [closure]) that provides the initial value | reduces the array (in reverse order) into a single value via the accumulator function:<br/>1st parameter: accumulated value ([`()`] initially)<br/>2nd parameter: array item<br/>3rd parameter: _(optional)_ offset index |
| `some`                    | [function pointer] to predicate (usually a [closure])                                                                                                                        | returns `true` if any item returns `true` when called with the predicate function:<br/>1st parameter: array item<br/>2nd parameter: _(optional)_ offset index                                                             |
| `all`                     | [function pointer] to predicate (usually a [closure])                                                                                                                        | returns `true` if all items return `true` when called with the predicate function:<br/>1st parameter: array item<br/>2nd parameter: _(optional)_ offset index                                                             |
| `sort`                    | [function pointer] to a comparison function (usually a [closure])                                                                                                            | sorts the array with a comparison function:<br/>1st parameter: first item<br/>2nd parameter: second item<br/>return value: `INT` < 0 if first < second, > 0 if first > second, 0 if first == second                       |


Use Custom Types With Arrays
---------------------------

To use a [custom type] with arrays, a number of array functions need to be manually implemented,
in particular `push`, `insert`, `pad` and the `+=` operator.  In addition, the `==` operator must be
implemented for the [custom type] in order to support the `in` operator which uses `==` to
compare elements.

See the section on [custom types] for more details.


Examples
--------

```rust,no_run
let y = [2, 3];             // y == [2, 3]

let y = [2, 3,];            // y == [2, 3]

y.insert(0, 1);             // y == [1, 2, 3]

y.insert(999, 4);           // y == [1, 2, 3, 4]

y.len == 4;

y[0] == 1;
y[1] == 2;
y[2] == 3;
y[3] == 4;

(1 in y) == true;           // use 'in' to test if an item exists in the array
(42 in y) == false;         // 'in' uses the '==' operator (which users can override)
                            // to check if the target item exists in the array

y[1] = 42;                  // y == [1, 42, 3, 4]

(42 in y) == true;

y.remove(2) == 3;           // y == [1, 42, 4]

y.len == 3;

y[2] == 4;                  // elements after the removed element are shifted

ts.list = y;                // arrays can be assigned completely (by value copy)

ts.list[1] == 42;

[1, 2, 3][0] == 1;          // indexing on array literal

fn abc() {
    [42, 43, 44]            // a function returning an array
}

abc()[0] == 42;

y.push(4);                  // y == [1, 42, 4, 4]

y += 5;                     // y == [1, 42, 4, 4, 5]

y.len == 5;

y.shift() == 1;             // y == [42, 4, 4, 5]

y.chop(3);                  // y == [4, 4, 5]

y.len == 3;

y.pop() == 5;               // y == [4, 4]

y.len == 2;

for item in y {             // arrays can be iterated with a 'for' statement
    print(item);
}

y.pad(6, "hello");          // y == [4, 4, "hello", "hello", "hello", "hello"]

y.len == 6;

y.truncate(4);              // y == [4, 4, "hello", "hello"]

y.len == 4;

y.clear();                  // y == []

y.len == 0;

let a = [42, 123, 99];

a.map(|v| v + 1);           // returns [43, 124, 100]

a.map(|v, i| v + i);        // returns [42, 124, 101]

a.filter(|v| v > 50);       // returns [123, 99]

a.filter(|v, i| i == 1);    // returns [123]

// Use a closure to provide the initial value
a.reduce(|sum, v| sum + v, || 0) == 264;

// Detect the initial value of '()'
a.reduce(
    |sum, v| if sum.type_of() == "()" { v } else { sum + v }
) == 264;

// Detect the initial value via index
a.reduce(|sum, v, i|
    if i == 0 { v } else { sum + v }
) == 264;

// Use a closure to provide the initial value
a.reduce_rev(|sum, v| sum + v, || 0) == 264;

// Detect the initial value of '()'
a.reduce_rev(
    |sum, v| if sum.type_of() == "()" { v } else { sum + v }
) == 264;

// Detect the initial value via index
a.reduce_rev(|sum, v, i|
    if i == 2 { v } else { sum + v }
) == 264;

a.some(|v| v > 50);         // returns true

a.some(|v, i| v < i);       // returns false

a.none(|v| v != 0);         // returns false

a.none(|v, i| v == i);      // returns true

a.all(|v| v > 50);          // returns false

a.all(|v, i| v > i);        // returns true

a.splice(1, 1, [1, 3, 2]);  // a == [42, 1, 3, 2, 99]

a.extract(1, 3);            // returns [1, 3, 2]

a.sort(|x, y| x - y);       // a == [1, 2, 3, 42, 99]

a.drain(|v| v <= 1);        // a == [2, 3, 42, 99]

a.drain(|v, i| i ≥ 3);     // a == [2, 3, 42]

a.retain(|v| v > 10);       // a == [42]

a.retain(|v, i| i > 0);     // a == []
```
