Safety and Protection Against DoS Attacks
========================================

{{#title Safety and Protection}}

{{#include ../links.md}}

For scripting systems open to untrusted user-land scripts, it is always best to limit the amount of
resources used by a script so that it does not consume more resources that it is allowed to.

The most important resources to watch out for are:

* **Memory**: A malicious script may continuously grow a [string], an [array], a [BLOB] or [object map] until all memory is consumed.

  It may also create a large [array] or [object map] literal that exhausts all memory during parsing.

* **CPU**: A malicious script may run an infinite tight loop that consumes all CPU cycles.

* **Time**: A malicious script may run indefinitely, thereby blocking the calling system which is waiting for a result.

* **Stack**: A malicious script may attempt an infinite recursive call that exhausts the call stack.

  Alternatively, it may create a degenerated deep expression with so many levels that the parser exhausts the call stack
  when parsing the expression; or even deeply-nested statement blocks, if nested deep enough.

  Another way to cause a stack overflow is to load a [self-referencing module][`import`].

* **Overflows**: A malicious script may deliberately cause numeric over-flows and/or under-flows, divide by zero, and/or
  create bad floating-point representations, in order to crash the system.

* **Files**: A malicious script may continuously [`import`] an external module within an infinite loop,
  thereby putting heavy load on the file-system (or even the network if the file is not local).

  Even when modules are not created from files, they still typically consume a lot of resources to load.

* **Data**: A malicious script may attempt to read from and/or write to data that it does not own. If this happens,
  it is a severe security breach and may put the entire system at risk.


_Don't Panic_ Guarantee &ndash; Any Panic is a Bug
-------------------------------------------------

![Don't Panic](https://upload.wikimedia.org/wikipedia/commons/thumb/6/6b/Don%27t_Panic.svg/320px-Don%27t_Panic.svg.png)

Rhai is designed to not bring down the host system, regardless of what a script may do to it.
This is a central design goal &ndash; Rhai provides a [_Don't Panic_](https://en.wikipedia.org/wiki/Phrases_from_The_Hitchhiker%27s_Guide_to_the_Galaxy#Don't_Panic) guarantee.

When using Rhai, any panic outside of API's with explicitly documented panic conditions is
considered a bug in Rhai and should be reported as such.


~~~admonish tip "OK, panic anyway &ndash; `unchecked`"

All the above safe-guards can be turned off via the [`unchecked`] feature, which disables all safety
checks (even fatal ones such as stack overflow, arithmetic overflow and division-by-zero).

This increases script evaluation performance somewhat, but at the expense of breaking the no-panic guarantee.

Under [`unchecked`], it is very possible for a malicious script to panic and bring down the host system.
~~~

~~~admonish warning "Beware of `internals`"

The [`internals`] feature allows third-party access to Rust internal data types and functions (for
example, the [`AST`] and related types).

This is usually a _Very Bad Idea™_ because:

* Messing up Rhai's internal data structures will easily create panics that bring down the host
  environment, violating the _Don't Panic_ guarantee.

* Allowing access to internal types may open up new attack vectors.

* Internal Rhai types and functions are volatile, so they may change from version to version and
  break code.

Use [`internals`] only if the operating environment has absolutely no safety concerns &ndash; you'd
be surprised how few scenarios this assumption holds.

One example of such an environment is a Rhai scripting [`Engine`] compiled to [WASM] where the
[`AST`] is further translated to include environment-specific modifications.
```
