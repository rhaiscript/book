Debugging Interface
===================

{{#include ../../links.md}}

For systems open to external user-created scripts, it is usually desirable to provide a _debugging_
experience to the user. The alternative is to provide a custom implementation of [`debug`] via
`Engine::on_debug` that traps debug output to show in a side panel, for example, which is actually
extremely simple.

Nevertheless, in some systems, it may not be convenient, or even possible, for the user to debug his
or her scripts simply via good-old [`print`] or [`debug`] statements &ndash; the system does not
have any facility for printed output, for instance.

Or the system may require more advanced debugging facilities than mere [`print`] statements &ndash;
such as [break-points].

For these advanced scenarios, Rhai contains a _Debugging_ interface, turned on via the [`debugging`]
feature (which implies the [`internals`] feature).

The debugging interface resides under the `debugger` sub-module.


Built-in Functions
-----------------

The following functions (defined in the [`DebuggingPackage`][built-in packages] but excluded if
using a [raw `Engine`]) provides runtime information for debugging purposes.

| Function     | Parameter(s) |      Not available under      | Description                                                                                                                                                 |
| ------------ | ------------ | :---------------------------: | ----------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `back_trace` | _none_       | [`no_function`], [`no_index`] | returns an [array] of [object maps] or [strings], each containing one level of [function] call;</br>returns an empty [array] if no [debugger] is registered |

```rust,no_run
// This recursive function prints its own call stack during each run
fn foo(x) {
    print(back_trace());        // prints the current call stack

    if x > 0 {
        foo(x - 1)
    }
}
```


Example
-------

The [`rhai-dbg`]({{repoHome}}/src/bin/rhai-dbg.rs) bin tool shows a simple example of
employing the debugging interface to create a debugger for Rhai scripts!

