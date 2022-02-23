Track Progress and Force-Termination
===================================

{{#include ../links.md}}

```admonish info.side.wide "Operations count vs. progress percentage"

_Operations count_ does not indicate the _amount_ of work already done &ndash; thus it is not real _progress_ tracking.

The real progress can be _estimated_ based on the expected number of operations in a typical run.
```

It is impossible to know when, or even whether, a script run will end
(a.k.a. the [Halting Problem](http://en.wikipedia.org/wiki/Halting_problem)).

When dealing with third-party untrusted scripts that may be malicious, in order to track evaluation
progress and force-terminate a script prematurely (for any reason), provide a closure to the
[`Engine`] via `Engine::on_progress`.

The closure passed to `Engine::on_progress` will be called once for every operation.

Progress tracking is disabled with the [`unchecked`] feature.


Example
-------

```rust,no_run
let mut engine = Engine::new();

engine.on_progress(|count| {    // parameter is number of operations already performed
    if count % 1000 == 0 {
        println!("{}", count);  // print out a progress log every 1,000 operations
    }
    None                        // return 'None' to continue running the script
                                // return 'Some(token)' to immediately terminate the script
});
```


### Function signature

The signature of the closure to pass to `Engine::on_progress` is as follows.

> `Fn(operations: u64) -> Option<Dynamic>`

### Return value

|     Value     | Effect                                                                           |
| :-----------: | -------------------------------------------------------------------------------- |
| `Some(token)` | terminate immediately, with `token` (a [`Dynamic`] value) as _termination token_ |
|    `None`     | continue script evaluation                                                       |

### Termination Token

```admonish info.side "Token"

The termination token is commonly used to provide information on the _reason_ behind the termination decision.
```

The [`Dynamic`] value returned is a _termination token_.

A script that is manually terminated returns with the error `EvalAltResult::ErrorTerminated(token, position)`
wrapping this value.

If the termination token is not needed, simply return `Some(Dynamic::UNIT)` to terminate the script
run with [`()`] as the token.
