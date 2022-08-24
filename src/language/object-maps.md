Object Maps
===========

{{#include ../links.md}}

```admonish tip.side "Safety"

Always limit the [maximum size of object maps].
```

Object maps are hash dictionaries. Properties are all [`Dynamic`] and can be freely added and retrieved.

The Rust type of a Rhai object map is `rhai::Map`.
Currently it is an alias to `BTreeMap<SmartString, Dynamic>`.

[`type_of()`] an object map returns `"map"`.

Object maps are disabled via the [`no_object`] feature.

~~~admonish question.small "Why `SmartString`?"

[`SmartString`] is used because most object map properties are short (at least shorter than 23 characters)
and ASCII-based, so they can usually be stored inline without incurring the cost of an allocation.
~~~

~~~admonish question.small "Why `BTreeMap` and not `HashMap`?"

The vast majority of object maps contain just a few properties.

`BTreeMap` performs significantly better than `HashMap` when the number of entries is small.
~~~


Literal Syntax
--------------

Object map literals are built within braces `#{` ... `}` with _name_`:`_value_ pairs separated by
commas `,`:

> `#{` _property_ `:` _value_`,` ... `,` _property_ `:` _value_ `}`
>
> `#{` _property_ `:` _value_`,` ... `,` _property_ `:` _value_ `,` `}`     `// trailing comma is OK`

The property _name_ can be a simple identifier following the same naming rules as [variables],
or a [string literal][literals] without interpolation.


Property Access Syntax
----------------------

### Dot notation

The _dot notation_ allows only property names that follow the same naming rules as [variables].

> _object_ `.` _property_

### Elvis notation

The [_Elvis notation_][elvis] is similar to the _dot notation_ except that it returns [`()`] if the object
itself is [`()`].

> `// returns () if object is ()`  
> _object_ `?.` _property_
>
> `// no action if object is ()`  
> _object_ `?.` _property_ `=` _value_ `;`

### Index notation

The _index notation_ allows setting/getting properties of arbitrary names (even the empty [string]).

> _object_ `[` _property_ `]`

### Non-existing property

```admonish tip.side.wide "Tip: Force error"

It is possible to force Rhai to return an `EvalAltResult:: ErrorPropertyNotFound` via
[`Engine:: set_fail_on_invalid_map_property`][options].
```

Trying to read a non-existing property returns [`()`] instead of causing an error.

This is similar to JavaScript where accessing a non-existing property returns `undefined`.

Use the [_Elvis operator_][elvis] (`?.`) to short-circuit further processing if the object is [`()`].

```rust
x.a.b.foo();        // <- error if 'x', 'x.a' or 'x.a.b' is ()

x.a.b = 42;         // <- error if 'x' or 'x.a' is ()

x?.a?.b?.foo();     // <- ok! returns () if 'x', 'x.a' or 'x.a.b' is ()

x?.a?.b = 42;       // <- ok even if 'x' or 'x.a' is ()
```


Built-in Functions
------------------

The following methods (defined in the [`BasicMapPackage`][built-in packages] but excluded if using a [raw `Engine`])
operate on object maps.

| Function                    | Parameter(s)                                                 | Description                                                                                                                                                           |
| --------------------------- | ------------------------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `get`                       | property name                                                | gets a copy of the value of a certain property ([`()`] if the property does not exist); behavior is not affected by [`Engine::fail_on_invalid_map_property`][options] |
| `set`                       | <ol><li>property name</li><li>new element</li></ol>          | sets a certain property to a new value (property is added if not already exists)                                                                                      |
| `len`                       | _none_                                                       | returns the number of properties                                                                                                                                      |
| `is_empty`                  | _none_                                                       | returns `true` if the object map is empty                                                                                                                             |
| `clear`                     | _none_                                                       | empties the object map                                                                                                                                                |
| `remove`                    | property name                                                | removes a certain property and returns it ([`()`] if the property does not exist)                                                                                     |
| `+=` operator, `mixin`      | second object map                                            | mixes in all the properties of the second object map to the first (values of properties with the same names replace the existing values)                              |
| `+` operator                | <ol><li>first object map</li><li>second object map</li></ol> | merges the first object map with the second                                                                                                                           |
| `==` operator               | <ol><li>first object map</li><li>second object map</li></ol> | are the two object maps the same (elements compared with the `==` operator, if defined)?                                                                              |
| `!=` operator               | <ol><li>first object map</li><li>second object map</li></ol> | are the two object maps different (elements compared with the `==` operator, if defined)?                                                                             |
| `fill_with`                 | second object map                                            | adds in all properties of the second object map that do not exist in the object map                                                                                   |
| `contains`, [`in`] operator | property name                                                | does the object map contain a property of a particular name?                                                                                                          |
| `keys`                      | _none_                                                       | returns an [array] of all the property names (in random order), not available under [`no_index`]                                                                      |
| `values`                    | _none_                                                       | returns an [array] of all the property values (in random order), not available under [`no_index`]                                                                     |
| `to_json`                   | _none_                                                       | returns a JSON representation of the object map ([`()`] is mapped to `null`, all other data types must be supported by JSON)                                          |


Examples
--------

```rust
let y = #{              // object map literal with 3 properties
    a: 1,
    bar: "hello",
    "baz!$@": 123.456,  // like JavaScript, you can use any string as property names...
    "": false,          // even the empty string!

    `hello`: 999,       // literal strings are also OK

    a: 42,              // <- syntax error: duplicated property name

    `a${2}`: 42,        // <- syntax error: property name cannot have string interpolation
};

y.a = 42;               // access via dot notation
y.a == 42;

y.baz!$@ = 42;          // <- syntax error: only proper variable names allowed in dot notation
y."baz!$@" = 42;        // <- syntax error: strings not allowed in dot notation
y["baz!$@"] = 42;       // access via index notation is OK

"baz!$@" in y == true;  // use 'in' to test if a property exists in the object map
("z" in y) == false;

ts.obj = y;             // object maps can be assigned completely (by value copy)
let foo = ts.list.a;
foo == 42;

let foo = #{ a:1, };    // trailing comma is OK

let foo = #{ a:1, b:2, c:3 }["a"];
let foo = #{ a:1, b:2, c:3 }.a;
foo == 1;

fn abc() {
    #{ a:1, b:2, c:3 }  // a function returning an object map
}

let foo = abc().b;
foo == 2;

let foo = y["a"];
foo == 42;

y.contains("a") == true;
y.contains("xyz") == false;

y.xyz == ();            // a non-existing property returns '()'
y["xyz"] == ();

y.len == ();            // an object map has no property getter function
y.len() == 3;           // method calls are OK

y.remove("a") == 1;     // remove property

y.len() == 2;
y.contains("a") == false;

for name in y.keys() {  // get an array of all the property names via 'keys'
    print(name);
}

for val in y.values() { // get an array of all the property values via 'values'
    print(val);
}

y.clear();              // empty the object map

y.len() == 0;
```


No Support for Property Getters
-------------------------------

In order not to affect the speed of accessing properties in an object map, new
[property getters][getters/setters] cannot be registered because they conflict with the syntax of
property access.

A [property getter][getters/setters] function registered via `Engine::register_get`, for example,
for a `Map` will never be found &ndash; instead, the property will be looked up in the object map.

Properties should be registered as _methods_ instead:

```rust
map.len                 // access property 'len', returns '()' if not found

map.len()               // 'len' method - returns the number of properties

map.keys                // access property 'keys', returns '()' if not found

map.keys()              // 'keys' method - returns array of all property names

map.values              // access property 'values', returns '()' if not found

map.values()            // 'values' method - returns array of all property values
```
