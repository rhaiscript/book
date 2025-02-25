Advanced Usage &ndash; Restore `NativeCallContext`
==================================================

{{#include ../links.md}}


The [`NativeCallContext`] type encapsulates the entire _context_ of a script up to the
particular point of the native Rust function call.

The data inside a [`NativeCallContext`] can be stored (as a type `NativeCallContextStore`) for later
use, when a new [`NativeCallContext`] can be constructed based on these stored data.

A reconstructed [`NativeCallContext`] acts almost the same as the original instance, so it is possible
to suspend the evaluation of a script, and to continue at a later time with a new
[`NativeCallContext`].

Doing so requires the [`internals`] feature to access internal APIs.

### Step 1: Store `NativeCallContext` data

```rust
// Store context for later use
let context_data = context.store_data();

// ... store 'context_data' somewhere ...
secret_database.push(context_data);
```

### Step 2: Restore `NativeCallContext`

```rust
// ... do something else ...

// Restore the context
let context_data = secret_database.get();

let new_context = context_data.create_context(&engine);
```
