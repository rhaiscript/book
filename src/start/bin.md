Packaged Utilities
==================

{{#include ../links.md}}

A number of Rhai-driven tools can be found in the `src/bin` directory:

|                       Tool                       | Description                                                 |
| :----------------------------------------------: | ----------------------------------------------------------- |
| [`rhai-repl`]({{repoHome}}/src/bin/rhai-repl.rs) | a simple REPL, interactively evaluate statements from stdin |
|  [`rhai-run`]({{repoHome}}/src/bin/rhai-run.rs)  | runs each filename passed to it as a Rhai script            |
|  [`rhai-dbg`]({{repoHome}}/src/bin/rhai-dbg.rs)  | the _Rhai Debugger_                                         |


Install Tools
-------------

To install these tools (with [`decimal`] and [`metadata`] support), use the following command:

```sh
cargo install --path . --bins  --features decimal,metadata
```

or specifically:

```sh
cargo install --path . --bin rhai-run  --features decimal,metadata
```


`rhai-repl` &ndash; The Rhai REPL Tool
-------------------------------------

`rhai-repl` is a particularly useful tool &ndash; it allows one to interactively try out
Rhai's language features in a standard REPL (**R**ead-**E**val-**P**rint **L**oop).

Filenames passed to it as command line arguments are run and loaded before the REPL starts.

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


`rhai-run` &ndash; The Rhai Runner
---------------------------------

Use `rhai-run` to run Rhai scripts.

Filenames passed to it as command line arguments are run in sequence.

### Example

The following command runs the scripts `script1.rhai`, `script2.rhai` and `script3.rhai` in order.

```sh
rhai-run script1.rhai script2.rhai script3.rhai
```


`rhai-dbg` &ndash; The Rhai Debugger
-----------------------------------

Use `rhai-dbg` to run Rhai scripts.

Filenames passed to it as command line arguments are run in sequence.

### Example

The following command debugs the script `my_script.rhai`.

```sh
rhai-dbg my_script.rhai
```


Running a Tool from Cargo
-------------------------

Tools can also be run with the following `cargo` command:

```sh
cargo run --bin {program_name}
```
