Packaged Utilities
==================

{{#include ../links.md}}

A number of Rhai-driven utility programs can be found in the `src/bin` directory:

|                 Utility program                  | Description                                                 |
| :----------------------------------------------: | ----------------------------------------------------------- |
| [`rhai-repl`]({{repoHome}}/src/bin/rhai-repl.rs) | a simple REPL, interactively evaluate statements from stdin |
|  [`rhai-run`]({{repoHome}}/src/bin/rhai-run.rs)  | runs each filename passed to it as a Rhai script            |


`rhai-repl` &ndash; The Rhai REPL Tool
-------------------------------------

`rhai-repl` is a particularly useful utility program &ndash; it allows one to interactively
try out Rhai's language features in a standard REPL (**R**ead-**E**val-**P**rint **L**oop).

Filenames passed to it as command line arguments are run and loaded before the REPL starts.

### Example

The following command first runs three scripts &ndash; `init1.rhai`, `init2.rhai` and
`init3.rhai` &ndash; loading the functions defined in each script into the _global_
namespace.

Then it enters an REPL, which can call the above functions freely.

```bash
rhai-repl init1.rhai init2.rhai init3.rhai
```


`rhai-run` &ndash; The Rhai Runner
---------------------------------

Use `rhai-run` to run Rhai scripts.

Filenames passed to it as command line arguments are run in sequence.

### Example

The following command runs the scripts `script1.rhai`, `script2.rhai` and `script3.rhai`
in order.

```bash
rhai-run script1.rhai script2.rhai script3.rhai
```


Running a Utility Program
-------------------------

Utilities can be run with the following command:

```bash
cargo run --bin {program_name}
```
