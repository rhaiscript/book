Object Maps
===========

{{#include ../links.md}}

Object maps are hash dictionaries. Properties are all [`Dynamic`] and can be freely added and retrieved.

The Rust type of a Rhai object map is `rhai::Map`.

[`type_of()`] an object map returns `"map"`.

Object maps are disabled via the [`no_object`] feature.

The maximum allowed size of an object map can be controlled via `Engine::set_max_map_size`
(see [maximum size of object maps]).


Object Map Literals
------------------

Object map literals are built within braces `#{` ... `}` (_name_ `:` _value_ syntax similar to Rust)
and separated by commas `,`:

> `#{` _property_ `:` _value_ `,` `...` `,` _property_ `:` _value_ `}`
>
> `#{` _property_ `:` _value_ `,` `...` `,` _property_ `:` _value_ `,` `}`  `// trailing comma is OK`

The property _name_ can be a simple variable name following the same
naming rules as [variables], or an arbitrary [string] literal.


Access Properties
-----------------

### Dot Notation

The _dot notation_ allows only property names that follow the same naming rules as [variables].

> _object_ `.` _property_

### Index Notation

The _index notation_ allows setting/getting properties of arbitrary names (even the empty [string]).

> _object_ `[` _property_ `]`

### Non-Existence

Trying to read a non-existing property returns [`()`] instead of causing an error.

This is similar to JavaScript where accessing a non-existing property returns `undefined`.


Built-in Functions
-----------------

The following methods (defined in the [`BasicMapPackage`][packages] but excluded if using a [raw `Engine`])
operate on object maps:

| Function               | Parameter(s)                                 | Description                                                                                                                              |
| ---------------------- | -------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------- |
| `has`, `in` operator   | property name                                | does the object map contain a property of a particular name?                                                                             |
| `len`                  | _none_                                       | returns the number of properties                                                                                                         |
| `clear`                | _none_                                       | empties the object map                                                                                                                   |
| `remove`               | property name                                | removes a certain property and returns it ([`()`] if the property does not exist)                                                        |
| `+=` operator, `mixin` | second object map                            | mixes in all the properties of the second object map to the first (values of properties with the same names replace the existing values) |
| `+` operator           | 1) first object map<br/>2) second object map | merges the first object map with the second                                                                                              |
| `==` operator          | 1) first object map<br/>2) second object map | are the two object map the same (elements compared with the `==` operator, if defined)?                                                  |
| `!=` operator          | 1) first object map<br/>2) second object map | are the two object map different (elements compared with the `==` operator, if defined)?                                                 |
| `fill_with`            | second object map                            | adds in all properties of the second object map that do not exist in the object map                                                      |
| `keys`                 | _none_                                       | returns an [array] of all the property names (in random order), not available under [`no_index`]                                         |
| `values`               | _none_                                       | returns an [array] of all the property values (in random order), not available under [`no_index`]                                        |


Examples
--------

```rust,no_run
let y = #{              // object map literal with 3 properties
    a: 1,
    bar: "hello",
    "baz!$@": 123.456,  // like JavaScript, you can use any string as property names...
    "": false,          // even the empty string!

    a: 42               // <- syntax error: duplicated property name
};

y.a = 42;               // access via dot notation
y.baz!$@ = 42;          // <- syntax error: only proper variable names allowed in dot notation
y."baz!$@" = 42;        // <- syntax error: strings not allowed in dot notation

y.a == 42;

y["baz!$@"] == 123.456; // access via index notation

"baz!$@" in y == true;  // use 'in' to test if a property exists in the object map
("z" in y) == false;

ts.obj = y;             // object maps can be assigned completely (by value copy)
let foo = ts.list.a;
foo == 42;

let foo = #{ a:1,};     // trailing comma is OK

let foo = #{ a:1, b:2, c:3 }["a"];
foo == 1;

fn abc() {
    ##{ a:1, b:2, c:3 } // a function returning an object map
}

let foo = abc().b;
foo == 2;

let foo = y["a"];
foo == 42;

y.has("a") == true;
y.has("xyz") == false;

y.xyz == ();            // a non-existing property returns '()'
y["xyz"] == ();

y.len() == 3;

y.remove("a") == 1;     // remove property

y.len() == 2;
y.has("a") == false;

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
------------------------------

In order not to affect the speed of accessing properties in an object map, new property
[getters][getters/setters] cannot be registered because they conflict with the syntax of
property access.

A property [getter][getters/setters] function registered via `Engine::register_get`, for example,
for a `Map` will never be found &ndash; instead, the property will be looked up in the object map.

Therefore, _method-call_ notation must be used for built-in properties:

```rust,no_run
map.len                 // access property 'len', returns '()' if not found

map.len()               // returns the number of properties

map.keys                // access property 'keys', returns '()' if not found

map.keys()              // returns array of all property names

map.values              // access property 'values', returns '()' if not found

map.values()            // returns array of all property values
```
