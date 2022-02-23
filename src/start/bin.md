Packaged Utilities
==================

{{#include ../links.md}}

A number of Rhai-driven tools can be found in the `src/bin` directory:

|                       Tool                       | Required feature(s) | Description            |
| :----------------------------------------------: | :-----------------: | ---------------------- |
|  [`rhai-run`]({{repoHome}}/src/bin/rhai-run.rs)  |                     | runs Rhai script files |
| [`rhai-repl`]({{repoHome}}/src/bin/rhai-repl.rs) |     `rustyline`     | a simple REPL tool     |
|  [`rhai-dbg`]({{repoHome}}/src/bin/rhai-dbg.rs)  |    [`debugging`]    | the _Rhai Debugger_    |

~~~admonish tip.small "Tip: `bin-features`"
Some bin tools require certain [features] and will not be built by default without those [features] set.

For convenience, a feature named [`bin-features`][features] is available which is a combination of
the following:

|    Feature    | Description                                 |
| :-----------: | ------------------------------------------- |
|  [`decimal`]  | support for [decimal][rust_decimal] numbers |
| [`metadata`]  | access [functions metadata]                 |
|   [`serde`]   | export [functions metadata] to JSON         |
| [`debugging`] | required by `rhai-dbg`                      |
|  `rustyline`  | required by `rhai-repl`                     |
~~~


Install Tools
-------------

To install all these tools (with full features), use the following command:

```sh
cargo install --path . --bins  --features bin-features
```

or specifically:

```sh
cargo install --path . --bin sample_app_to_run  --features bin-features
```


Run a Tool from Cargo
---------------------

Tools can also be run with the following `cargo` command:

```sh
cargo run --features bin-features --bin sample_app_to_run
```


Tools List
----------

~~~admonish example "`rhai-repl` &ndash; The Rhai REPL Tool"

`rhai-repl` is a particularly useful tool &ndash; it allows one to interactively try out
Rhai's language features in a standard REPL (**R**ead-**E**val-**P**rint **L**oop).

Filenames passed to it as command line arguments are run and loaded as Rhai scripts before the REPL starts.

### Test functions

The following test functions are pre-registered, via `Engine::register_fn`, into `rhai-repl`.
They are intended for testing purposes.

| Function                             | Description                                |
| ------------------------------------ | ------------------------------------------ |
| `test(x: i64, y: i64)`               | returns a string with both numbers         |
| `test(x: &mut i64, y: i64, z: &str)` | displays the parameters and add `y` to `x` |

### Example

The following command first runs three scripts &ndash; `init1.rhai`, `init2.rhai` and `init3.rhai` &ndash;
loading the functions defined in each script into the _global_ namespace.

Then it enters an REPL, which can call the above functions freely.

```sh
rhai-repl init1.rhai init2.rhai init3.rhai
```
~~~

~~~admonish danger "`rhai-run` &ndash; The Rhai Runner"

Use `rhai-run` to run Rhai scripts.

Filenames passed to it as command line arguments are run in sequence as Rhai scripts.

### Example

The following command runs the scripts `script1.rhai`, `script2.rhai` and `script3.rhai` in order.

```sh
rhai-run script1.rhai script2.rhai script3.rhai
```
~~~

~~~admonish bug "`rhai-dbg` &ndash; The Rhai Debugger"

Use `rhai-dbg` to debug a Rhai script.

Filename passed to it will be loaded as a Rhai script for debugging.

### Example

The following command debugs the script `my_script.rhai`.

```sh
rhai-dbg my_script.rhai
```
~~~
