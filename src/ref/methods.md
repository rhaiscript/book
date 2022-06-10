Methods
=======

Data types may have _methods_ that can be called:

> _object_ `.` _method_ `(` _parameters_ ... `)`

A runtime error is raised if the appropriate method does not exist for the object's data type.

The [_Elvis_ operator](https://en.wikipedia.org/wiki/Elvis_operator) can be used to short-circuit
the method call when the object itself is `()`.

> _object_ `?.` _method_ `(` _parameters_ ... `)`

In the above, the _method_ is never called if _object_ is `()`.
