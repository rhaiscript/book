Indexing
========

```admonish tip.side "Tip: Non-integer index"

Some data types take an index that is not an integer.
For example, [object map](object-maps.md) indices are [strings](strings-chars.md).
```

Some data types, such as [arrays](arrays.md), can be _indexed_ via a Rust-like syntax:

> _object_ `[` _index_ `]`
>
> _object_ `[` _index_ `]` `=` _value_ `;`

Usually, a runtime error is raised if the index value is out of bounds or does not exist for the
object's data type.


Elvis Notation
--------------

The [_Elvis notation_](https://en.wikipedia.org/wiki/Elvis_operator) is similar except that it
returns `()` if the object itself is `()`.

> `// returns () if object is ()`  
> _object_ `?[` _index_ `]`
>
> `// no action if object is ()`  
> _object_ `?[` _index_ `]` `=` _value_ `;`
