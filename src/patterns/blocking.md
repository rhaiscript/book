Blocking/Async Function Calls
=============================

{{#include ../links.md}}


```admonish info "Usage scenarios"

* A system's API contains async functions.
```

```admonish danger.small "Warning: Async and scripting don't mix well"

Otherwise, you get the [_Callback Hell_](https://en.wiktionary.org/wiki/callback_hell)
which is JavaScript before all the async extensions.
```

```admonish abstract "Key concepts"

* This pattern is based upon the _[Multi-Threaded Synchronization](multi-threading.md)_ pattern.

* An independent thread is used to run the scripting [`Engine`].

* An MPSC channel (or any other appropriate synchronization primitive) is used to send function call
  arguments, packaged as a message, to another Rust thread that will perform the actual async calls.

* Results are marshaled back to the [`Engine`] thread via another MPSC channel.
```

```admonish info.small "See also"

See the _[Multi-Threaded Synchronization](multi-threading.md)_ pattern.
```


Implementation
--------------

1. Spawn a thread to run the scripting [`Engine`].  Usually the [`sync`] feature is
   _NOT_ used for this pattern.

2. Spawn another thread (the `worker` thread) that can perform the actual async calls in Rust.
   This thread may actually be the main thread of the program.

3. Create a pair of MPSC channels (named `command` and `reply` below) for full-duplex
   communications between the two threads.

4. Register async API function to the [`Engine`] with a closure that captures the MPSC end-points.

5. If there are more than one async function, the receive end-point on the `reply` channel can simply be cloned.
   The send end-point on the `command` channel can be wrapped in an `Arc<Mutex<Channel>>` for shared access.

6. In the async function, the name of the function and call arguments are serialized into JSON
   (or any appropriate message format) and sent to `command` channel, where they'll be removed
   by the `worker` thread and the appropriate async function called.

7. The [`Engine`] blocks on the function call, waiting for a reply message on the `reply` channel.

8. When the async function call complete on the `worker` thread, the result is sent back to
   the [`Engine`] thread via the `reply` channel.

9.  After the result is obtained from the `reply` channel, the [`Engine`] returns it as the return value
   of the function call, ending the block and continuing evaluation.
