Object-Oriented Programming (OOP)
=================================

{{#include ../links.md}}

Rhai does not have _objects_ per se and is not object-oriented (in the traditional sense),
but it is possible to _simulate_ object-oriented programming.

```admonish question.small "To OOP or not to OOP, that is the question."

Regardless of whether object-oriented programming (OOP) should be treated as a pattern or
an _anti-pattern_ (the programming world is split 50-50 on this), there are always users who
would like to write Rhai in "the OOP way."

Rust itself is not object-oriented in the traditional sense; JavaScript also isn't, but that didn't
prevent generations of programmers trying to shoehorn a class-based inheritance system onto it.

So... as soon as Rhai gained in usage, way way before version 1.0, PR's started coming in to make
it possible to write Rhai in "the OOP way."
```


Use Object Maps to Simulate OOP
-------------------------------

Rhai's [object maps] has [special support for OOP]({{rootUrl}}/language/object-maps-oop.md).

| Rhai concept                                          | Maps to OOP |
| ----------------------------------------------------- | :---------: |
| [Object maps]                                         |   objects   |
| [Object map] properties holding values                | properties  |
| [Object map] properties that hold [function pointers] |   methods   |

When a property of an [object map] is called like a method function, and if it happens to hold a
valid [function pointer] (perhaps defined via an [anonymous function] or more commonly as a [closure]),
then the call will be dispatched to the actual function with `this` binding to the
[object map] itself.


Use Closures to Define Methods
------------------------------

[Anonymous functions] or [closures] defined as values for [object map] properties take on a
syntactic shape which resembles very closely that of class methods in an OOP language.

Closures also _[capture][automatic currying]_ variables from the defining environment, which is a
very common language feature.  Capturing is accomplished via a feature called _[automatic currying]_
and can be turned off via the [`no_closure`] feature.

```rust
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
-------------------------------------

The `fill_with` method of [object maps] can be conveniently used to _polyfill_ default method
implementations from a _base class_, as per OOP lingo.

Do not use the `mixin` method because it _overwrites_ existing fields.

```rust
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

```rust
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

```admonish warning.small "Warning: Exporting OOP objects"
Objects containing anonymous functions cannot be exported into separated modules:
Anonymous functions de-sugar into function pointers. However, those cannot
point to a function from a module. Thus, it is mandatory to define all objects into
the script that will be loaded by the engine.

However, there are still (questionnable) ways to export objects.
You could, for example, create a macro that appends the content of a module to the
main script.

~~~rust
// my-module.rhai

const my_object = #{
    value: 5,
    inc: || this.value += 1
    dec: || this.value -= 1
};
~~~

~~~rust
#IMPORT "my-module" // `#IMPORT` is our new macro, it copies the content of "my-module.rhai" to this file.

my_object.inc(); // `value` is incremented to 6.
~~~

Beware that since the contents of the module are copied into the main script, this could lead to unexpected behavior.
```
