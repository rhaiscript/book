Timestamps
==========

Timestamps are provided by the via the `timestamp` function.

[`type_of()`](type-of.md) a timestamp returns `"timestamp"`.


Built-in Functions
------------------

The following methods operate on timestamps.

| Function                      | Parameter(s)                                                | Description                                                           |
| ----------------------------- | ----------------------------------------------------------- | --------------------------------------------------------------------- |
| `elapsed` method and property | _none_                                                      | returns the number of seconds since the timestamp                     |
| `+` operator                  | number of seconds to add                                    | returns a new timestamp with a specified number of seconds added      |
| `+=` operator                 | number of seconds to add                                    | adds a specified number of seconds to the timestamp                   |
| `-` operator                  | number of seconds to subtract                               | returns a new timestamp with a specified number of seconds subtracted |
| `-=` operator                 | number of seconds to subtract                               | subtracts a specified number of seconds from the timestamp            |
| `-` operator                  | <ol><li>later timestamp</li><li>earlier timestamp</li></ol> | returns the number of seconds between the two timestamps              |

The following time-related functions are also available.

| Function | Parameter(s)               | Description                                                 |
| -------- | -------------------------- | ----------------------------------------------------------- |
| `sleep`  | number of seconds to sleep | blocks the current thread for a specified number of seconds |


Examples
--------

```rust
let now = timestamp();

// Do some lengthy operation...

if now.elapsed > 30.0 {
    print("takes too long (over 30 seconds)!")
}
```
