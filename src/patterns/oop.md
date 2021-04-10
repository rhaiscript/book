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

```rust , no_run
let factor = 1;

// Define the object
let obj = #{
        data: 0,                            // object field
        increment: |x| this.data += x,      // 'this' binds to 'obj'
        update: |x| this.data = x * factor, // 'this' binds to 'obj', 'factor' is captured
        action: || print(this.data)         // 'this' binds to 'obj'
    };

// Use the object
obj.increment(1);
obj.action();                               // prints 1

obj.update(42);
obj.action();                               // prints 42

factor = 2;

obj.update(42);
obj.action();                               // prints 84
```


Simulating Inheritance with Polyfills
------------------------------------

The `fill_with` method of [object maps] can be conveniently used to _polyfill_ default
method implementations from a _base class_, as per OOP lingo.

Do not use the `mixin` method because it _overwrites_ existing fields.

```rust , no_run
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


Prototypical Inheritance via Mixin
----------------------------------

Some languages like JavaScript has _prototypical_ inheritance, which bases inheritance on a
_prototype_ object.

It is possible to simulate this form of inheritance using [object maps], leveraging the fact that,
in Rhai, all values are cloned and there are no pointers. This significantly simplifies coding logic.

```rust , no_run
// Define prototype 'class'

const PrototypeClass = #{
    field: 42,

    get_field: || this.field,
    set_field: |x| this.field = x
};

// Create instances of the 'class'

let obj1 = PrototypeClass;                  // a copy of 'PrototypeClass'

obj1.get_field() == 42;

let obj2 = PrototypeClass;                  // a copy of 'PrototypeClass'

obj2.mixin(#{                               // override fields and methods
    field: 1,
    get_field: || this.field * 2
};

obj2.get_field() == 2;

let obj2 = PrototypeClass + #{              // compact syntax with '+'
    field: 1,
    get_field: || this.field * 2
};

obj2.get_field() == 2;

// Inheritance chain

const ParentClass = #{
    field: 123,
    new_field: 0,
    action: || print(this.new_field * this.field)
};

const ChildClass = #{
    action: || {
        this.field = this.new_field;
        this.new_field = ();
    }
}

let obj3 = PrototypeClass + ParentClass + ChildClass;

// Alternate formulation

const ParentClass = PrototypeClass + #{
    field: 123,
    new_field: 0,
    action: || print(this.new_field * this.field)
};

const ChildClass = ParentClass + #{
    action: || {
        this.field = this.new_field;
        this.new_field = ();
    }
}

let obj3 = ChildClass;                      // a copy of 'ChildClass'
```
