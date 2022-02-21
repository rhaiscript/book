Track Progress and Force-Termination
===================================

{{#include ../links.md}}

```admonish info.side-wide "Operations count vs. progress percentage"

_Operations count_ does not indicate the _amount_ of work already done &ndash; thus it is not real _progress_ tracking.

The real progress can be _estimated_ based on the expected number of operations in a typical run.
```

It is impossible to know when, or even whether, a script run will end
(a.k.a. the [Halting Problem](http://en.wikipedia.org/wiki/Halting_problem)).

When dealing with third-party untrusted scripts that may be malicious, in order to track evaluation
progress and force-terminate a script prematurely (for any reason), provide a closure to the
[`Engine`] via `Engine::on_progress`.

Progress tracking is disabled with the [`unchecked`] feature.

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

The closure passed to `Engine::on_progress` will be called once for every operation.

| Return value of closure | Effect                                                                         |
| :---------------------: | ------------------------------------------------------------------------------ |
|      `Some(token)`      | terminate immediately, with `token` (a [`Dynamic`] value) as termination token |
|         `None`          | continue script evaluation                                                     |


Termination Token
-----------------

The [`Dynamic`] value returned by the closure for `Engine::on_progress` is a _termination token_.

A script that is manually terminated returns with the error `EvalAltResult::ErrorTerminated(token, position)`
wrapping this value.

The termination token is commonly used to provide information on the _reason_ or _source_
behind the termination decision.

If the termination token is not needed, simply return `Some(Dynamic::UNIT)` to terminate the script
run with [`()`] as the token.
