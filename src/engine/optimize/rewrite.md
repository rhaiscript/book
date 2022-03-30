Compound Assignment Rewrite
===========================

{{#include ../../links.md}}

```admonish info.side "Avoid cloning"

Arguments passed as value are always cloned.
```

Usually, a _compound assignment_ (e.g. `+=` for append) takes a mutable first parameter
(i.e. `&mut`) while the corresponding simple [operator] (i.e. `+`) does not.

The script optimizer rewrites normal assignments into _compound assignments_ wherever possible in
order to avoid unnecessary cloning.

```rust
let big = create_some_very_big_type();

big = big + 1;
//    ^ 'big' is cloned here

// The above is equivalent to:
let temp_value = big + 1;
big = temp_value;

big += 1;           // <- 'big' is NOT cloned
```

~~~admonish warning.small "Warning: Simple references only"

Only _simple variable references_ are optimized.

No [_common sub-expression elimination_](https://en.wikipedia.org/wiki/Common_subexpression_elimination)
is performed by Rhai.

```rust
x = x + 1;          // <- this statement...

x += 1;             // <- ... is rewritten to this

x[y] = x[y] + 1;    // <- but this is not,
                    //    so MUCH slower...

x[y] += 1;          // <- ... than this
```
~~~
