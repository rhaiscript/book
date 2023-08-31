Arrays
======

{{#include ../links.md}}

```admonish tip.side "Safety"

Always limit the [maximum size of arrays].
```

Arrays are first-class citizens in Rhai.

All elements stored in an array are [`Dynamic`], and the array can freely grow or shrink with
elements added or removed.

The Rust type of a Rhai array is `rhai::Array` which is an alias to `Vec<Dynamic>`.

[`type_of()`] an array returns `"array"`.

Arrays are disabled via the [`no_index`] feature.


Literal Syntax
--------------

Array literals are built within square brackets `[` ... `]` and separated by commas `,`:

> `[` _value_`,` _value_`,` ... `,` _value_ `]`
>
> `[` _value_`,` _value_`,` ... `,` _value_ `,` `]`     `// trailing comma is OK`


Element Access Syntax
---------------------

### From beginning

Like C, arrays are accessed with zero-based, non-negative integer indices:

> _array_ `[` _index position from 0 to (length−1)_ `]`

### From end

A _negative_ position accesses an element in the array counting from the _end_, with −1 being the
_last_ element.

> _array_ `[` _index position from −1 to −length_ `]`


Built-in Functions
------------------

The following methods (mostly defined in the [`BasicArrayPackage`][built-in packages] but excluded
when using a [raw `Engine`]) operate on arrays.

| Function                       | Parameter(s)                                                                                                                                                   | Description                                                                                                                                                                                                                                                                                                               |
| ------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `get`                          | position, counting from end if < 0                                                                                                                             | gets a copy of the element at a certain position ([`()`] if the position is not valid)                                                                                                                                                                                                                                    |
| `set`                          | <ol><li>position, counting from end if < 0</li><li>new element</li></ol>                                                                                       | sets a certain position to a new value (no effect if the position is not valid)                                                                                                                                                                                                                                           |
| `push`, `+=` operator          | element to append (not an array)                                                                                                                               | appends an element to the end                                                                                                                                                                                                                                                                                             |
| `append`, `+=` operator        | array to append                                                                                                                                                | concatenates the second array to the end of the first                                                                                                                                                                                                                                                                     |
| `+` operator                   | <ol><li>first array</li><li>second array</li></ol>                                                                                                             | concatenates the first array with the second                                                                                                                                                                                                                                                                              |
| `==` operator                  | <ol><li>first array</li><li>second array</li></ol>                                                                                                             | are two arrays the same (elements compared with the `==` operator, if defined)?                                                                                                                                                                                                                                           |
| `!=` operator                  | <ol><li>first array</li><li>second array</li></ol>                                                                                                             | are two arrays different (elements compared with the `==` operator, if defined)?                                                                                                                                                                                                                                          |
| `insert`                       | <ol><li>position, counting from end if < 0, end if ≥ length</li><li>element to insert</li></ol>                                                                | inserts an element at a certain position                                                                                                                                                                                                                                                                                  |
| `pop`                          | _none_                                                                                                                                                         | removes the last element and returns it ([`()`] if empty)                                                                                                                                                                                                                                                                 |
| `shift`                        | _none_                                                                                                                                                         | removes the first element and returns it ([`()`] if empty)                                                                                                                                                                                                                                                                |
| `extract`                      | <ol><li>start position, counting from end if < 0, end if ≥ length</li><li>_(optional)_ number of elements to extract, none if ≤ 0, to end if omitted</li></ol> | extracts a portion of the array into a new array                                                                                                                                                                                                                                                                          |
| `extract`                      | [range] of elements to extract, from beginning if ≤ 0, to end if ≥ length                                                                                      | extracts a portion of the array into a new array                                                                                                                                                                                                                                                                          |
| `remove`                       | position, counting from end if < 0                                                                                                                             | removes an element at a particular position and returns it ([`()`] if the position is not valid)                                                                                                                                                                                                                          |
| `reverse`                      | _none_                                                                                                                                                         | reverses the array                                                                                                                                                                                                                                                                                                        |
| `len` method and property      | _none_                                                                                                                                                         | returns the number of elements                                                                                                                                                                                                                                                                                            |
| `is_empty` method and property | _none_                                                                                                                                                         | returns `true` if the array is empty                                                                                                                                                                                                                                                                                      |
| `pad`                          | <ol><li>target length</li><li>element to pad</li></ol>                                                                                                         | pads the array with an element to at least a specified length                                                                                                                                                                                                                                                             |
| `clear`                        | _none_                                                                                                                                                         | empties the array                                                                                                                                                                                                                                                                                                         |
| `truncate`                     | target length                                                                                                                                                  | cuts off the array at exactly a specified length (discarding all subsequent elements)                                                                                                                                                                                                                                     |
| `chop`                         | target length                                                                                                                                                  | cuts off the head of the array, leaving the tail at exactly a specified length                                                                                                                                                                                                                                            |
| `split`                        | <ol><li>array</li><li>position to split at, counting from end if < 0, end if ≥ length</li></ol>                                                                | splits the array into two arrays, starting from a specified position                                                                                                                                                                                                                                                      |
| `for_each`                     | [function pointer] for processing elements                                                                                                                     | run through each element in the array in order, binding each to `this` and calling the processing function taking the following parameters: <ol><li>`this`: array element</li><li>_(optional)_ index position</li></ol>                                                                                                   |
| `drain`                        | [function pointer] to predicate (usually a [closure])                                                                                                          | removes all elements (returning them) that return `true` when called with the predicate function taking the following parameters (if none, the array element is bound to `this`):<ol><li>array element</li><li>_(optional)_ index position</li></ol>                                                                      |
| `drain`                        | <ol><li>start position, counting from end if < 0, end if ≥ length</li><li>number of elements to remove, none if ≤ 0</li></ol>                                  | removes a portion of the array, returning the removed elements as a new array                                                                                                                                                                                                                                             |
| `drain`                        | [range] of elements to remove, from beginning if ≤ 0, to end if ≥ length                                                                                       | removes a portion of the array, returning the removed elements as a new array                                                                                                                                                                                                                                             |
| `retain`                       | [function pointer] to predicate (usually a [closure])                                                                                                          | removes all elements (returning them) that do not return `true` when called with the predicate function taking the following parameters (if none, the array element is bound to `this`):<ol><li>array element</li><li>_(optional)_ index position</li></ol>                                                               |
| `retain`                       | <ol><li>start position, counting from end if < 0, end if ≥ length</li><li>number of elements to retain, none if ≤ 0</li></ol>                                  | retains a portion of the array, removes all other elements and returning them as a new array                                                                                                                                                                                                                              |
| `retain`                       | [range] of elements to retain, from beginning if ≤ 0, to end if ≥ length                                                                                       | retains a portion of the array, removes all other bytes and returning them as a new array                                                                                                                                                                                                                                 |
| `splice`                       | <ol><li>start position, counting from end if < 0, end if ≥ length</li><li>number of elements to remove, none if ≤ 0</li><li>array to insert</li></ol>          | replaces a portion of the array with another (not necessarily of the same length as the replaced portion)                                                                                                                                                                                                                 |
| `splice`                       | <ol><li>[range] of elements to remove, from beginning if ≤ 0, to end if ≥ length</li><li>array to insert</li></ol>                                             | replaces a portion of the array with another (not necessarily of the same length as the replaced portion)                                                                                                                                                                                                                 |
| `filter`                       | [function pointer] to predicate (usually a [closure])                                                                                                          | constructs a new array with all elements that return `true` when called with the predicate function taking the following parameters (if none, the array element is bound to `this`):<ol><li>array element</li><li>_(optional)_ index position</li></ol>                                                                   |
| `contains`, [`in`] operator    | element to find                                                                                                                                                | does the array contain an element? The `==` operator (if defined) is used to compare [custom types]                                                                                                                                                                                                                       |
| `index_of`                     | <ol><li>element to find (not a [function pointer])</li><li>_(optional)_ start position, counting from end if < 0, end if ≥ length</li></ol>                    | returns the position of the first element in the array that equals the supplied element (using the `==` operator, if defined), or −1 if not found</li></ol>                                                                                                                                                               |
| `index_of`                     | <ol><li>[function pointer] to predicate (usually a [closure])</li><li>_(optional)_ start position, counting from end if < 0, end if ≥ length</li></ol>         | returns the position of the first element in the array that returns `true` when called with the predicate function, or −1 if not found:<ol><li>array element (if none, the array element is bound to `this`)</li><li>_(optional)_ index position</li></ol>                                                                |
| `find`                         | <ol><li>[function pointer] to predicate (usually a [closure])</li><li>_(optional)_ start position, counting from end if < 0, end if ≥ length</li></ol>         | returns the first element in the array that returns `true` when called with the predicate function, or [`()`] if not found:<ol><li>array element (if none, the array element is bound to `this`)</li><li>_(optional)_ index position</li></ol>                                                                            |
| `find_map`                     | <ol><li>[function pointer] to predicate (usually a [closure])</li><li>_(optional)_ start position, counting from end if < 0, end if ≥ length</li></ol>         | returns the first non-[`()`] value of the first element in the array when called with the predicate function, or [`()`] if not found:<ol><li>array element (if none, the array element is bound to `this`)</li><li>_(optional)_ index position</li></ol>                                                                  |
| `dedup`                        | _(optional)_ [function pointer] to predicate (usually a [closure]); if omitted, the `==` operator is used, if defined                                          | removes all but the first of _consecutive_ elements in the array that return `true` when called with the predicate function (non-consecutive duplicates are _not_ removed):<br/>1st & 2nd parameters: two elements in the array                                                                                           |
| `map`                          | [function pointer] to conversion function (usually a [closure])                                                                                                | constructs a new array with all elements mapped to the result of applying the conversion function taking the following parameters (if none, the array element is bound to `this`):<ol><li>array element</li><li>_(optional)_ index position</li></ol>                                                                     |
| `reduce`                       | <ol><li>[function pointer] to accumulator function (usually a [closure])</li><li>_(optional)_ the initial value</li></ol>                                      | reduces the array into a single value via the accumulator function taking the following parameters (if the second parameter is omitted, the array element is bound to `this`):<ol><li>accumulated value ([`()`] initially)</li><li>`this`: array element</li><li>_(optional)_ index position</li></ol>                    |
| `reduce_rev`                   | <ol><li>[function pointer] to accumulator function (usually a [closure])</li><li>_(optional)_ the initial value</li></ol>                                      | reduces the array (in reverse order) into a single value via the accumulator function taking the following parameters (if the second parameter is omitted, the array element is bound to `this`):<ol><li>accumulated value ([`()`] initially)</li><li>`this`: array element</li><li>_(optional)_ index position</li></ol> |
| `zip`                          | <ol><li>array to zip</li><li>[function pointer] to conversion function (usually a [closure])</li></ol>                                                         | constructs a new array with all element pairs from two arrays mapped to the result of applying the conversion function taking the following parameters:<ol><li>first array element</li><li>second array element</li><li>_(optional)_ index position</li></ol>                                                             |
| `some`                         | [function pointer] to predicate (usually a [closure])                                                                                                          | returns `true` if any element returns `true` when called with the predicate function taking the following parameters (if none, the array element is bound to `this`):<ol><li>array element</li><li>_(optional)_ index position</li></ol>                                                                                  |
| `all`                          | [function pointer] to predicate (usually a [closure])                                                                                                          | returns `true` if all elements return `true` when called with the predicate function taking the following parameters (if none, the array element is bound to `this`):<ol><li>array element</li><li>_(optional)_ index position</li></ol>                                                                                  |
| `sort`                         | [function pointer] to a comparison function (usually a [closure])                                                                                              | sorts the array with a comparison function taking the following parameters:<ol><li>first element</li><li>second element<br/>return value: `INT` < 0 if first < second, > 0 if first > second, 0 if first == second</li></ol>                                                                                              |
| `sort`                         | _none_                                                                                                                                                         | sorts a _homogeneous_ array containing only elements of the same comparable built-in type (`INT`, `FLOAT`, [`Decimal`][rust_decimal], [string], [character], `bool`, [`()`])                                                                                                                                              |


```admonish tip.small "Tip: Use custom types with arrays"

To use a [custom type] with arrays, a number of functions need to be manually implemented,
in particular the `==` operator in order to support the [`in`] operator which uses `==` (via the
`contains` method) to compare elements.

See the section on [custom types] for more details.
```


Examples
--------

```rust
let y = [2, 3];             // y == [2, 3]

let y = [2, 3,];            // y == [2, 3]

y.insert(0, 1);             // y == [1, 2, 3]

y.insert(999, 4);           // y == [1, 2, 3, 4]

y.len == 4;

y[0] == 1;
y[1] == 2;
y[2] == 3;
y[3] == 4;

(1 in y) == true;           // use 'in' to test if an element exists in the array

(42 in y) == false;         // 'in' uses the 'contains' function, which uses the
                            // '==' operator (that users can override)
                            // to check if the target element exists in the array

y.contains(1) == true;      // the above de-sugars to this

y[1] = 42;                  // y == [1, 42, 3, 4]

(42 in y) == true;

y.remove(2) == 3;           // y == [1, 42, 4]

y.len == 3;

y[2] == 4;                  // elements after the removed element are shifted

ts.list = y;                // arrays can be assigned completely (by value copy)

ts.list[1] == 42;

[1, 2, 3][0] == 1;          // indexing on array literal

[1, 2, 3][-1] == 3;         // negative position counts from the end

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

for element in y {             // arrays can be iterated with a 'for' statement
    print(element);
}

y.pad(6, "hello");          // y == [4, 4, "hello", "hello", "hello", "hello"]

y.len == 6;

y.truncate(4);              // y == [4, 4, "hello", "hello"]

y.len == 4;

y.clear();                  // y == []

y.len == 0;

// The examples below use 'a' as the master array

let a = [42, 123, 99];

a.for_each(|| this *= 2);

a == [84, 246, 198];

a.for_each(|i| this /= 2);

a == [42, 123, 99];

a.map(|v| v + 1);           // returns [43, 124, 100]

a.map(|| this + 1);         // returns [43, 124, 100]

a.map(|v, i| v + i);        // returns [42, 124, 101]

a.filter(|v| v > 50);       // returns [123, 99]

a.filter(|| this > 50);     // returns [123, 99]

a.filter(|v, i| i == 1);    // returns [123]

a.filter("is_odd");         // returns [123, 99]

a.filter(Fn("is_odd"));     // <- previous statement is equivalent to this...

a.filter(|v| is_odd(v));    // <- or this

a.some(|v| v > 50);         // returns true

a.some(|| this > 50);       // returns true

a.some(|v, i| v < i);       // returns false

a.all(|v| v > 50);          // returns false

a.all(|| this > 50);        // returns false

a.all(|v, i| v > i);        // returns true

// Reducing - initial value provided directly
a.reduce(|sum| sum + this, 0) == 264;

// Reducing - initial value provided directly
a.reduce(|sum, v| sum + v, 0) == 264;

// Reducing - initial value is '()'
a.reduce(
    |sum, v| if sum.type_of() == "()" { v } else { sum + v }
) == 264;

// Reducing - initial value has index position == 0
a.reduce(|sum, v, i|
    if i == 0 { v } else { sum + v }
) == 264;

// Reducing in reverse - initial value provided directly
a.reduce_rev(|sum| sum + this, 0) == 264;

// Reducing in reverse - initial value provided directly
a.reduce_rev(|sum, v| sum + v, 0) == 264;

// Reducing in reverse - initial value is '()'
a.reduce_rev(
    |sum, v| if sum.type_of() == "()" { v } else { sum + v }
) == 264;

// Reducing in reverse - initial value has index position == 0
a.reduce_rev(|sum, v, i|
    if i == 2 { v } else { sum + v }
) == 264;

// In-place modification

a.splice(1..=1, [1, 3, 2]); // a == [42, 1, 3, 2, 99]

a.extract(1..=3);           // returns [1, 3, 2]

a.sort(|x, y| y - x);       // a == [99, 42, 3, 2, 1]

a.sort();                   // a == [1, 2, 3, 42, 99]

a.drain(|v| v <= 1);        // a == [2, 3, 42, 99]

a.drain(|v, i| i ≥ 3);      // a == [2, 3, 42]

a.retain(|v| v > 10);       // a == [42]

a.retain(|v, i| i > 0);     // a == []
```
