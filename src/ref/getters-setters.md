Properties
==========

Data types typically expose properties, which can be accessed in a Rust-like syntax:

> _object_ `.` _property_
>
> _object_ `.` _property_ `=` _value_ `;`

A runtime error is raised if the property does not exist for the object's data type.


Elvis Operator
--------------

The [_Elvis operator_](https://en.wikipedia.org/wiki/Elvis_operator) can be used to short-circuit
processing if the object itself is `()`.

> `// returns () if object is ()`  
> _object_ `?.` _property_
>
> `// no action if object is ()`  
> _object_ `?.` _property_ `=` _value_ `;`
