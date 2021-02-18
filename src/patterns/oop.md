Object-Oriented Programming (OOP)
================================

{{#include ../links.md}}

Rhai does not have _objects_ per se, but it is possible to _simulate_ object-oriented programming.


Use Object Maps to Simulate OOP
------------------------------

Rhai's [object maps] has [special support for OOP]({{rootUrl}}/language/object-maps-oop.md).

| Rhai concept                                          | Maps to OOP |
| ----------------------------------------------------- | :---------: |
| [Object maps]                                         |   objects   |
| [Object map] properties holding values                | properties  |
| [Object map] properties that hold [function pointers] |   methods   |

When a property of an [object map] is called like a method function, and if it happens to hold
a valid [function pointer] (perhaps defined via an [anonymous function] or more commonly as a [closure]),
then the call will be dispatched to the actual function with `this` binding to the [object map] itself.


Use Closures to Define Methods
-----------------------------

[Anonymous functions] or [closures] defined as values for [object map] properties take on
a syntactic shape which resembles very closely that of class methods in an OOP language.

Closures also _[capture][automatic currying]_ variables from the defining environment, which is a very
common language feature.  Capturing is accomplished via a feature called _[automatic currying]_ and
can be turned off via the [`no_closure`] feature.


Examples
--------

```rust,no_run
let factor = 1;

// Define the object
let obj = #{
        data: 0,                             // object field
        increment: |x| this.data += x,       // 'this' binds to 'obj'
        update: |x| this.data = x * factor,  // 'this' binds to 'obj', 'factor' is captured
        action: || print(this.data)          // 'this' binds to 'obj'
    };

// Use the object
obj.increment(1);
obj.action();                                // prints 1

obj.update(42);
obj.action();                                // prints 42

factor = 2;

obj.update(42);
obj.action();                                // prints 84
```


Simulating Inheritance With Mixin
--------------------------------

The `fill_with` method of [object maps] can be conveniently used to _polyfill_ default
method implementations from a _base class_, as per OOP lingo.

Do not use the `mixin` method because it _overwrites_ existing fields.

```rust,no_run
// Define base class
let BaseClass = #{
    factor: 1,
    data: 42,

    get_data: || this.data * 2,
    update: |x| this.data += x * this.factor
};

let obj = #{
    // Override base class field
    factor: 100,

    // Override base class method
    // Notice that the base class can also be accessed, if in scope
    get_data: || this.call(BaseClass.get_data) * 999,
}

// Polyfill missing fields/methods
obj.fill_with(BaseClass);

// By this point, 'obj' has the following:
//
// #{
//      factor: 100
//      data: 42,
//      get_data: || this.call(BaseClass.get_data) * 999,
//      update: |x| this.data += x * this.factor
// }

// obj.get_data() => (this.data (42) * 2) * 999
obj.get_data() == 83916;

obj.update(1);

obj.data == 142
```
