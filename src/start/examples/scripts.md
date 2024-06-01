Example Scripts
===============

{{#include ../../links.md}}

Language Feature Scripts
------------------------

There are also a number of examples scripts that showcase Rhai's features, all in the `scripts` directory:

| Script                                                            | Description                                                                                   |
| ----------------------------------------------------------------- | --------------------------------------------------------------------------------------------- |
| [`array.rhai`]({{repoHome}}/scripts/array.rhai)                   | [arrays] example                                                                              |
| [`assignment.rhai`]({{repoHome}}/scripts/assignment.rhai)         | [variable] declarations                                                                       |
| [`comments.rhai`]({{repoHome}}/scripts/comments.rhai)             | just regular [comments]                                                                       |
| [`doc-comments.rhai`]({{repoHome}}/scripts/doc-comments.rhai)     | [doc-comments] example                                                                        |
| [`for1.rhai`]({{repoHome}}/scripts/for1.rhai)                     | [`for`] loops                                                                                 |
| [`for2.rhai`]({{repoHome}}/scripts/for2.rhai)                     | [`for`] loops with [array] iterations                                                         |
| [`for3.rhai`]({{repoHome}}/scripts/for3.rhai)                     | [`for`] loops with [closures]                                                                 |
| [`function_decl1.rhai`]({{repoHome}}/scripts/function_decl1.rhai) | a [function] without parameters                                                               |
| [`function_decl2.rhai`]({{repoHome}}/scripts/function_decl2.rhai) | a [function] with two parameters                                                              |
| [`function_decl3.rhai`]({{repoHome}}/scripts/function_decl3.rhai) | a [function] with many parameters                                                             |
| [`function_decl4.rhai`]({{repoHome}}/scripts/function_decl4.rhai) | a [function] acting as a [method]({{rootUrl}}/language/fn-method.md)                          |
| [`function_decl5.rhai`]({{repoHome}}/scripts/function_decl5.rhai) | multiple [functions] as [methods]({{rootUrl}}/language/fn-method.md) for different data types |
| [`if1.rhai`]({{repoHome}}/scripts/if1.rhai)                       | [`if`] example                                                                                |
| [`if2.rhai`]({{repoHome}}/scripts/if2.rhai)                       | [`if`]-expression example                                                                     |
| [`loop.rhai`]({{repoHome}}/scripts/loop.rhai)                     | count-down [`loop`] in Rhai, emulating a [`do`] ... `while` loop                              |
| [`module.rhai`]({{repoHome}}/scripts/module.rhai)                 | import a script file as a module                                                              |
| [`oop.rhai`]({{repoHome}}/scripts/oop.rhai)                       | simulate [object-oriented programming (OOP)][OOP] with [closures]                             |
| [`op1.rhai`]({{repoHome}}/scripts/op1.rhai)                       | just simple addition                                                                          |
| [`op2.rhai`]({{repoHome}}/scripts/op2.rhai)                       | simple addition and multiplication                                                            |
| [`op3.rhai`]({{repoHome}}/scripts/op3.rhai)                       | change evaluation order with parenthesis                                                      |
| [`string.rhai`]({{repoHome}}/scripts/string.rhai)                 | [string] operations, including _interpolation_                                                |
| [`strings_map.rhai`]({{repoHome}}/scripts/strings_map.rhai)       | [string] and [object map] operations                                                          |
| [`switch.rhai`]({{repoHome}}/scripts/switch.rhai)                 | [`switch`] example                                                                            |
| [`while.rhai`]({{repoHome}}/scripts/while.rhai)                   | [`while`] loop                                                                                |


Benchmark Scripts
-----------------

The following scripts are for benchmarking the speed of Rhai:

| Scripts                                                   | Description                                                                            |
| --------------------------------------------------------- | -------------------------------------------------------------------------------------- |
| [`speed_test.rhai`]({{repoHome}}/scripts/speed_test.rhai) | a simple application to measure the speed of Rhai's interpreter (1 million iterations) |
| [`primes.rhai`]({{repoHome}}/scripts/primes.rhai)         | use Sieve of Eratosthenes to find all primes smaller than a limit                      |
| [`fibonacci.rhai`]({{repoHome}}/scripts/fibonacci.rhai)   | calculate the n-th Fibonacci number using a really dumb algorithm                      |
| [`mat_mul.rhai`]({{repoHome}}/scripts/mat_mul.rhai)       | matrix multiplication test to measure the speed of multi-dimensional array access      |


Run Example Scripts
-------------------

The [`rhai-run`]({{rootUrl}}/bin.md) utility can be used to run Rhai scripts:

```sh
cargo run --bin rhai-run scripts/any_script.rhai
```
