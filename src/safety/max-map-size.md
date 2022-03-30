Maximum Size of Object Maps
==========================

{{#include ../links.md}}

Rhai by default does not limit how large (i.e. the number of properties) an [object map] can be.

This can be changed via `Engine::set_max_map_size`, with zero being unlimited (the default).

A script attempting to create an [object map] literal with more properties than the maximum will terminate with a parse error.

Any script operation that produces an [object map] with more properties than the maximum also terminates the script with an error result.

This check can be disabled via the [`unchecked`] feature for higher performance (but higher risks as well).

```rust
let mut engine = Engine::new();

engine.set_max_map_size(500);       // allow object maps with only up to 500 properties

engine.set_max_map_size(0);         // allow unlimited object maps
```


~~~admonish danger "Maximum size"

Be conservative when setting a maximum limit and always consider the fact that a registered function
may grow an [object map]'s size without Rhai noticing until the very end.

For instance, the built-in `+` operator for [object maps] concatenates two [object maps] together to
form one larger [object map]; if both [object maps] are _slightly_ below the maximum size limit, the
resultant [object map] may be almost _twice_ the maximum size.

As a malicious script may create a deeply-nested [object map] which consumes huge amounts of memory
while each individual [object map] still stays under the maximum size limit, Rhai also _recursively_
adds up the sizes of all [strings], [arrays] and [object maps] contained within each [object map] to
make sure that the _aggregate_ sizes of none of these data structures exceed their respective
maximum size limits (if any).

```rust
// Small, innocent object map...
let small_map: #{ x: 42 };          // 1-deep... 1 item, 1 object map

// ... becomes huge when multiplied!
small_map.y = small_map;            // 2-deep... 2 items, 2 object maps
small_map.y = small_map;            // 3-deep... 4 items, 4 object maps
small_map.y = small_map;            // 4-deep... 8 items, 8 object maps
small_map.y = small_map;            // 5-deep... 16 items, 16 object maps
          :
          :
small_map.y = small_map;            // <- Rhai raises an error somewhere here
small_map.y = small_map;            //    when the TOTAL number of items in
small_map.y = small_map;            //    the entire array tree exceeds limit

// Or this abomination...
let map = #{ x: 42 };

loop {
    map.x = map;    // <- only 1 item, but infinite number of object maps
}
```
~~~
