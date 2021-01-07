Example Scripts
==============

{{#include ../../links.md}}

Language Feature Scripts
-----------------------

There are also a number of examples scripts that showcase Rhai's features, all in the `scripts` directory:

| Script                                                            | Description                                                                                 |
| ----------------------------------------------------------------- | ------------------------------------------------------------------------------------------- |
| [`array.rhai`]({{repoTree}}/scripts/array.rhai)                   | [arrays]                                                                                    |
| [`assignment.rhai`]({{repoTree}}/scripts/assignment.rhai)         | variable declarations                                                                       |
| [`comments.rhai`]({{repoTree}}/scripts/comments.rhai)             | just comments                                                                               |
| [`for1.rhai`]({{repoTree}}/scripts/for1.rhai)                     | [`for`]({{rootUrl}}/language/for.md) loops                                                  |
| [`for2.rhai`]({{repoTree}}/scripts/for2.rhai)                     | [`for`]({{rootUrl}}/language/for.md) loops on [arrays]                                      |
| [`function_decl1.rhai`]({{repoTree}}/scripts/function_decl1.rhai) | a [function] without parameters                                                             |
| [`function_decl2.rhai`]({{repoTree}}/scripts/function_decl2.rhai) | a [function] with two parameters                                                            |
| [`function_decl3.rhai`]({{repoTree}}/scripts/function_decl3.rhai) | a [function] with many parameters                                                           |
| [`if1.rhai`]({{repoTree}}/scripts/if1.rhai)                       | [`if`]({{rootUrl}}/language/if.md) example                                                  |
| [`loop.rhai`]({{repoTree}}/scripts/loop.rhai)                     | count-down [`loop`]({{rootUrl}}/language/loop.md) in Rhai, emulating a `do` .. `while` loop |
| [`module.rhai`]({{repoTree}}/scripts/module.rhai)                 | import a script file as a module                                                            |
| [`oop.rhai`]({{repoTree}}/scripts/oop.rhai)                       | simulate [object-oriented programming (OOP)][OOP] with [closures]                           |
| [`op1.rhai`]({{repoTree}}/scripts/op1.rhai)                       | just simple addition                                                                        |
| [`op2.rhai`]({{repoTree}}/scripts/op2.rhai)                       | simple addition and multiplication                                                          |
| [`op3.rhai`]({{repoTree}}/scripts/op3.rhai)                       | change evaluation order with parenthesis                                                    |
| [`string.rhai`]({{repoTree}}/scripts/string.rhai)                 | [string] operations                                                                         |
| [`strings_map.rhai`]({{repoTree}}/scripts/strings_map.rhai)       | [string] and [object map] operations                                                        |
| [`while.rhai`]({{repoTree}}/scripts/while.rhai)                   | [`while`]({{rootUrl}}/language/while.md) loop                                               |


Benchmark Scripts
----------------

The following scripts are for benchmarking the speed of Rhai:

| Scripts                                                   | Description                                                                            |
| --------------------------------------------------------- | -------------------------------------------------------------------------------------- |
| [`speed_test.rhai`]({{repoTree}}/scripts/speed_test.rhai) | a simple application to measure the speed of Rhai's interpreter (1 million iterations) |
| [`primes.rhai`]({{repoTree}}/scripts/primes.rhai)         | use Sieve of Eratosthenes to find all primes smaller than a limit                      |
| [`fibonacci.rhai`]({{repoTree}}/scripts/fibonacci.rhai)   | calculate the n-th Fibonacci number using a really dumb algorithm                      |
| [`mat_mul.rhai`]({{repoTree}}/scripts/mat_mul.rhai)       | matrix multiplication test to measure the speed of multi-dimensional array access      |


Running Example Scripts
----------------------

The [`rhai-run`](../bin.md) utility can be used to run Rhai scripts:

```bash
cargo run --bin rhai-run scripts/any_script.rhai
```
