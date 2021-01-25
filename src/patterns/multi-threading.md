Multi-Threaded Synchronization
=============================

{{#include ../links.md}}


Usage Scenarios
---------------

* A system needs to communicate with an [`Engine`] running in a separate thread.

* Multiple [`Engine`]s running in separate threads need to coordinate/synchronize with each other.


Key Concepts
------------

* An MPSC channel (or any other appropriate synchronization primitive) is used to send/receive
  messages to/from an [`Engine`] running in a separate thread.

* An API is registered with the [`Engine`] that is essentially _blocking_ until synchronization is achieved.


Example
-------

```rust
use rhai::{Engine, RegisterFn};

fn main() {
    // Channel: Script -> Master
    let (tx_script, rx_master) = std::sync::mpsc::channel();
    // Channel: Master -> Script
    let (tx_master, rx_script) = std::sync::mpsc::channel();

    // Spawn thread with Engine
    std::thread::spawn(move || {
        // Create Engine
        let mut engine = Engine::new();

        // Register API
        // Notice that the API functions are blocking
        engine
            .register_fn("get", move || rx_script.recv().unwrap())
            .register_fn("put", move |v: i64| tx_script.send(v).unwrap());

        // Run script
        engine
            .consume(
                r#"
                    print("Starting script loop...");

                    loop {
                        // The following call blocks until there is data
                        // in the channel
                        let x = get();
                        print("Script Read: " + x);

                        x += 1;

                        print("Script Write: " + x);

                        // The following call blocks until the data
                        // is successfully sent to the channel
                        put(x);
                    }
                "#,
            )
            .unwrap();
    });

    // This is the main processing thread

    println!("Starting main loop...");

    let mut value = 0_i64;

    while value < 10 {
        println!("Value: {}", value);
        // Send value to script
        tx_master.send(value).unwrap();
        // Receive value from script
        value = rx_master.recv().unwrap();
    }
}
```


Considerations for [`sync`]
--------------------------

`std::mpsc::Sender` and `std::mpsc::Receiver` are not `Sync`, therefore they cannot be used in
registered functions if the [`sync`] feature is enabled.

In that situation, it is possible to wrap the `Sender` and `Receiver` each in a `Mutex`,
which makes them `Sync`. This, however, incurs the additional overhead of locking and unlocking
the `Mutex` during every function call, which is technically not necessary because there are no
other references to them.


Regarding Async Functions
-------------------------

The example above highlights the fact that Rhai scripts can call any Rust function,
including ones that are _blocking_.

However, Rhai is essentially a blocking, single-threaded engine.
Therefore it does _not_ provide an async API.

That means, although it is simple to use Rhai within a multi-threading environment where blocking a
thread is acceptable or even expected, it is currently not possible to call _async_ functions
within Rhai scripts because there is no mechanism in [`Engine`] to wrap the state of the call stack
inside a future.

Fortunately an [`Engine`] is re-entrant so it can be shared among many async tasks.
It is usually possible to split a script into multiple parts to avoid having to call async functions.

Creating an [`Engine`] is also relatively cheap (extremely cheap if creating a [raw `Engine`]),
so it is also a valid pattern to spawn a new [`Engine`] instance for each task.
