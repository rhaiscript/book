Special Support for OOP via Object Maps
=======================================

{{#title Object Maps and OOP}}

[Object maps](object-maps.md) can be used to simulate object-oriented programming (OOP) by storing
data as properties and methods as properties holding [function pointers](fn-ptr.md).

If an [object map](object-maps.md)'s property holds a [function pointer](fn-ptr.md), the property
can simply be called like a normal method in method-call syntax.

This is a _short-hand_ to avoid the more verbose syntax of using the `call` function keyword.

When a property holding a [function pointer](fn-ptr.md) or a [closure](fn-closure.md) is called like
a method, it is replaced as a method call on the [object map](object-maps.md) itself.

```rust
let obj = #{
                data: 40,
                action: || this.data += x    // 'action' holds a closure
           };

obj.action(2);                               // calls the function pointer with 'this' bound to 'obj'

obj.call(obj.action, 2);                     // <- the above de-sugars to this

obj.data == 42;

// To achieve the above with normal function pointer call will fail.

fn do_action(map, x) { map.data += x; }      // 'map' is a copy

obj.action = Fn("do_action");

obj.action.call(obj, 2);                     // a copy of 'obj' is passed by value

obj.data == 42;                              // 'obj.data' is not changed
```
