Dynamic Value Tag
=================

{{#include ../links.md}}

Each [`Dynamic`] value can contain a _tag_ that is `i16` and can contain any arbitrary data.

The _tag_ defaults to zero.

It is an error to set a tag to a value beyond the bounds of `i16`.


Examples
--------

```rust , no_run
let x = 42;

x.tag == 0;         // tag defaults to zero

x.tag = 0xab;       // set tag value

set_tag(x, 0xab);   // 'set_tag' function also works

x.tag == 171;       // get updated tag value

x.tag() == 171;     // method also works

tag(x) == 171;      // function call style also works

let y = x;

y.tag == 171;       // the tag is copied across assignment

y.tag = 0xabcd;     // runtime error: 0xabcd is too large for 'i16'
```


Practical Applications
----------------------

Attaching arbitrary information together with a value has a lot of practical uses.

### Identify code path

For example, it is easy to attach an ID number to a value to indicate how or why that value is
originally set.

This is tremendously convenient for debugging purposes where it is necessary to figure out which
code path a particular value went through.

After the script is verified, all tag assignment statements can simply be removed.

```js
const ROUTE1 = 1;
const ROUTE2 = 2;
const ROUTE3 = 3;
const ERROR_ROUTE = 9;

fn some_complex_calculation(x) {
    let result;

    if some_complex_condition(x) {
        result = 42;
        result.tag = ROUTE1;        // record route #1
    } else if some_other_very_complex_condition(x) == 1 {
        result = 123;
        result.tag = ROUTE2;        // record route #2
    } else if some_non_understandable_calculation(x) > 0 {
        result = 0;
        result.tag = ROUTE3;        // record route #3
    } else {
        result = -1;
        result.tag = ERROR_ROUTE;   // record error
    }

    result  // this value now contains the tag
}

let my_result = some_complex_calculation(key);

// The code path that 'my_result' went through is now in its tag.

// It is now easy to trace how 'my_result' gets its final value.

print(`Result = ${my_result} and reason = ${my_result.tag}`);
```

### Identify code conditions

The tag value may also contain a _bit-field_ of up to 16 individual bits, indicating up to 16 logic
conditions that contributed to the value.

Again, after the script is verified, all tag assignment statements can simply be removed.

```js

fn some_complex_calculation(x) {
    let result = 42;

    // Check first condition
    if some_complex_condition() {
        result += 1;
        result.tag |= 0b0001;
    }

    // Check second condition
    if some_other_very_complex_condition(x) == 1 {
        result *= 10;
        result.tag |= 0b0010;
    }

    // Check third condition
    if some_non_understandable_calculation(x) > 0 {
        result -= 42;
        result.tag |= 0b0100;
    }

    // Check result
    if result > 100 {
        result = 0;
        result.tag = 0b1000;
    }
}

let my_result = some_complex_calculation(key);

// The tag of 'my_result' now contains a bit-field indicating
// the result of each condition.

// It is now easy to trace how 'my_result' gets its final value.

print(`Result = ${my_result}`);
print(`First condition = ${(my_result.tag & 0b0001) != 0}`);
print(`Second condition = ${(my_result.tag & 0b0010) != 0}`);
print(`Third condition = ${(my_result.tag & 0b0100) != 0}`);
print(`Result check = ${(my_result.tag & 0b1000) != 0}`);
```
