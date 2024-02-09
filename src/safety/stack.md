Limiting Stack Usage
====================

{{#include ../links.md}}


Most O/S differentiates between _heap_ and _stack_ memory.

Usually the stack (around 1MB) is much smaller than the heap (multiple MB's or even GB's).

Therefore, it is possible for a carefully-crafted script to consume all available stack memory (such
as deeply-nested expressions) and crash the host system, even though there is ample heap memory
available.


Calculate Stack Usage
---------------------

Some O/S's provide system calls to get the stack size and/or amount of free stack memory, but these
are in the minority.

In order to determine the amount of stack memory actually used, it is necessary to perform some
pointer arithmetic.

The trick is to get the address of a stack-allocated variable in the very beginning and compare it
to the address of another variable.

```rust
// Create a variable on the stack.
let stack_base_ref = Dynamic::UNIT;
// Get a pointer to it.
let stack_base: *const Dynamic = &stack_base_ref;

// ... do a lot of work here ...

// Create another variable on the stack.
let stack_top = Dynamic::UNIT;
// Get a pointer to it.
let stack_top: *const Dynamic = &stack_top;

let usage = unsafe { stack_top.offset_from(stack_base) };
```

```admonish question.small "Negative values"

In many cases, the amount of stack memory used is actually _negative_
(meaning that the base variable is in a higher memory address than the current variable).

That is because, for many architectures, the stack grows _downwards_ and the heap
grows _upwards_ in order to maximize memory usage efficiency.
```

During Evaluation
-----------------

To prevent _stack-overflow_ failures, provide a closure to [`Engine::on_progress`][progress] to
track stack usage and force-terminate a malicious script before it can bring down the host system.

```rust
let mut engine = Engine::new();

const MAX_STACK: usize = 100 * 1024;    // 10KB

// Create a variable on the stack.
let stack_base_ref = Dynamic::UNIT;
// Get a pointer to it.
let stack_base: *const Dynamic = &stack_base_ref;

engine.on_progress(move |_| {
    // Create another variable on the stack.
    let stack_top = Dynamic::UNIT;
    // Get a pointer to it.
    let stack_top: *const Dynamic = &stack_top;

    let usage = unsafe { stack_base.offset_from(stack_top) };

    if usage > MAX_STACK {
        // Terminate the script 
        Some(Dynamic::UNIT)
    } else {
        // Continue
        None
    }
});
```


During Parsing
--------------

A malicious script can be carefully crafted such that it consumes all stack memory during the
parsing stage.

Protect against this by via a closure to [`Engine::on_parse_token`][token remap filter].

```rust
let mut engine = Engine::new();

const MAX_STACK: usize = 100 * 1024;    // 10KB

// Create a variable on the stack.
let stack_base_ref = Dynamic::UNIT;
// Get a pointer to it.
let stack_base: *const Dynamic = &stack_base_ref;

engine.on_parse_token(|token, _, _| {
    // Create another variable on the stack.
    let stack_top = Dynamic::UNIT;
    // Get a pointer to it.
    let stack_top: *const Dynamic = &stack_top;

    let usage = unsafe { stack_base.offset_from(stack_top) };

    if usage > MAX_STACK {
        // Terminate parsing
        Token::LexError(
            LexError::Runtime("stack-overflow".into()).into()
        )
    } else {
        // Continue
        token
    }
});
```
