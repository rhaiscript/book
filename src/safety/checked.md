Safety Checks
=============

{{#include ../links.md}}


Checked Arithmetic
------------------

By default, all arithmetic calculations in Rhai are _checked_, meaning that the script terminates
with a runtime error whenever it detects a numeric over-flow/under-flow condition or an invalid
floating-point operation.

Scripts under normal builds of Rhai never crash the host system &ndash; any panic is a bug.

This checking can be turned off via the [`unchecked`] feature for higher performance
(but higher risks as well).

```rust no_run
let x = 1_000_000_000_000;

x * x;      // Normal build: runtime error: multiplication overflow

x * x;      // 'unchecked' build: panic!

x / 0;      // Normal build: runtime error: division by zero

x / 0;      // 'unchecked' build: panic!
```


Other Safety Checks
-------------------

In addition to arithmetic overflows etc., there are many other safety checks performed by Rhai
at runtime. [`unchecked`] turns them **all** off as well, such as...

### [Infinite loops][maximum number of operations]

```rust no_run
// Normal build: runtime error: exceeds maximum number of operations
loop { foo(); }

// 'unchecked' build: never terminates!
loop { foo(); }
```

### [Infinite recursion][maximum call stack depth]

```rust no_run
fn foo() { foo(); }

foo();      // Normal build: runtime error: exceeds maximum stack depth

foo();      // 'unchecked' build: panic due to stack overflow!
```

### [Gigantic data structures][maximum size of arrays]

```rust no_run
let x = [];

// Normal build: runtime error: array exceeds maximum size
loop { x += 42; }

// 'unchecked' build: panic due to out-of-memory!
loop { x += 42; }
```

### Improper range iteration

```rust no_run
// Normal build: runtime error: zero step
for x in range(0, 10, 0) { ... }

// 'unchecked' build: never terminates!
for x in range(0, 10, 0) { ... }

// Normal build: empty range
for x in range(0, 10, -1) { ... }

// 'unchecked' build: panic due to numeric underflow!
for x in range(0, 10, -1) { ... }
```
