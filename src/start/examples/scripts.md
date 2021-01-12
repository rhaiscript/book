Example Scripts
==============

{{#include ../../links.md}}

Language Feature Scripts
-----------------------

There are also a number of examples scripts that showcase Rhai's features, all in the `scripts` directory:

| Script                                                            | Description                                                                                 |
| ----------------------------------------------------------------- | ------------------------------------------------------------------------------------------- |
| [`array.rhai`]({{repoHome}}/scripts/array.rhai)                   | [arrays]                                                                                    |
| [`assignment.rhai`]({{repoHome}}/scripts/assignment.rhai)         | variable declarations                                                                       |
| [`comments.rhai`]({{repoHome}}/scripts/comments.rhai)             | just comments                                                                               |
| [`for1.rhai`]({{repoHome}}/scripts/for1.rhai)                     | [`for`]({{rootUrl}}/language/for.md) loops                                                  |
| [`for2.rhai`]({{repoHome}}/scripts/for2.rhai)                     | [`for`]({{rootUrl}}/language/for.md) loops on [arrays]                                      |
| [`function_decl1.rhai`]({{repoHome}}/scripts/function_decl1.rhai) | a [function] without parameters                                                             |
| [`function_decl2.rhai`]({{repoHome}}/scripts/function_decl2.rhai) | a [function] with two parameters                                                            |
| [`function_decl3.rhai`]({{repoHome}}/scripts/function_decl3.rhai) | a [function] with many parameters                                                           |
| [`if1.rhai`]({{repoHome}}/scripts/if1.rhai)                       | [`if`]({{rootUrl}}/language/if.md) example                                                  |
| [`loop.rhai`]({{repoHome}}/scripts/loop.rhai)                     | count-down [`loop`]({{rootUrl}}/language/loop.md) in Rhai, emulating a `do` .. `while` loop |
| [`module.rhai`]({{repoHome}}/scripts/module.rhai)                 | import a script file as a module                                                            |
| [`oop.rhai`]({{repoHome}}/scripts/oop.rhai)                       | simulate [object-oriented programming (OOP)][OOP] with [closures]                           |
| [`op1.rhai`]({{repoHome}}/scripts/op1.rhai)                       | just simple addition                                                                        |
| [`op2.rhai`]({{repoHome}}/scripts/op2.rhai)                       | simple addition and multiplication                                                          |
| [`op3.rhai`]({{repoHome}}/scripts/op3.rhai)                       | change evaluation order with parenthesis                                                    |
| [`string.rhai`]({{repoHome}}/scripts/string.rhai)                 | [string] operations                                                                         |
| [`strings_map.rhai`]({{repoHome}}/scripts/strings_map.rhai)       | [string] and [object map] operations                                                        |
| [`while.rhai`]({{repoHome}}/scripts/while.rhai)                   | [`while`]({{rootUrl}}/language/while.md) loop                                               |


Benchmark Scripts
----------------

The following scripts are for benchmarking the speed of Rhai:

| Scripts                                                   | Description                                                                            |
| --------------------------------------------------------- | -------------------------------------------------------------------------------------- |
| [`speed_test.rhai`]({{repoHome}}/scripts/speed_test.rhai) | a simple application to measure the speed of Rhai's interpreter (1 million iterations) |
| [`primes.rhai`]({{repoHome}}/scripts/primes.rhai)         | use Sieve of Eratosthenes to find all primes smaller than a limit                      |
| [`fibonacci.rhai`]({{repoHome}}/scripts/fibonacci.rhai)   | calculate the n-th Fibonacci number using a really dumb algorithm                      |
| [`mat_mul.rhai`]({{repoHome}}/scripts/mat_mul.rhai)       | matrix multiplication test to measure the speed of multi-dimensional array access      |


Running Example Scripts
----------------------

The [`rhai-run`](../bin.md) utility can be used to run Rhai scripts:

```bash
cargo run --bin rhai-run scripts/any_script.rhai
```
