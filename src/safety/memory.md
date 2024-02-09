Limiting Memory Usage
=====================

{{#include ../links.md}}


During Evaluation
-----------------

To prevent _out-of-memory_ failures, provide a closure to [`Engine::on_progress`][progress] to track
memory usage and force-terminate a malicious script before it can bring down the host system.

Most O/S provides system calls to obtain the current memory usage of the process.

```rust
let mut engine = Engine::new();

const MAX_MEMORY: usize = 10 * 1024 * 1024;   // 10MB

engine.on_progress(|_| {
    // Call a system function to obtain the current memory usage
    let memory_usage = get_current_progress_memory_usage();

    if memory_usage > MAX_MEMORY {
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

A malicious script can be carefully crafted such that it consumes all available memory during the
parsing stage.

Protect against this by via a closure to [`Engine::on_parse_token`][token remap filter].

```rust
let mut engine = Engine::new();

const MAX_MEMORY: usize = 10 * 1024 * 1024;   // 10MB

engine.on_parse_token(|token, _, _| {
    // Call a system function to obtain the current memory usage
    let memory_usage = get_current_progress_memory_usage();

    if memory_usage > MAX_MEMORY {
        // Terminate parsing
        Token::LexError(
            LexError::Runtime("out of memory".into()).into()
        )
    } else {
        // Continue
        token
    }
});
```
