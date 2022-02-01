Implement a Debugging Server
============================

Sometimes it is desirable to embed a debugging _server_ inside the application such that an external
debugger interface can connect to the application's running instance at runtime.

This way, when scripts are run within the application, it is easy for an external interface to debug
those scripts as they run.

Such connections may take the form of any communication channel, for example a TCP/IP connection, a
named pipe, or an MPSC channel.


Example
-------

### Server side

The following example assumes bi-direction, blocking messaging channels, such as a WebSocket
connection, with a server that accepts connections and creates those channels.

```rust,no_run
use rhai::debugger::{ASTNode, DebuggerCommand};

let mut engine = Engine::new();

engine.register_debugger(
    // Use the initialization callback to set up the communications channel
    // and listen to it
    || {
        // Create server that will listen to requests
        let mut server = MyCommServer::new();
        server.listen("localhost:8080");

        // Wrap it up in a shared locked cell so it can be 'Clone'
        let server = Rc::new(RefCell::new(server));

        // Store the channel in the debugger state
        Dynamic::from(server)
    },
    // Trigger the server during each debugger stop point
    |context, event, node, source, pos| {
        // Get the state
        let mut state = debugger.state_mut();

        // Get the server
        let mut server = state.write_lock::<MyCommServer>().unwrap();

        // Send the event to the server - blocking call
        server.send_message(...);

        // Receive command - blocking call
        match server.receive_message() {
            None => DebuggerCommand::StepOver,
            // Decode command
            Ok(...) => { ... }
            Ok(...) => { ... }
            Ok(...) => { ... }
            Ok(...) => { ... }
            Ok(...) => { ... }
                :
                :
        }
    }
);
```

### Client side

The client can be any system that can work with WebSockets for messaging.

```js
// Connect to the application's debugger
let webSocket = new WebSocket("wss://localhost:8080");

webSocket.on_message = (event) => {
    let msg = JSON.parse(event.data);

    switch msg.type {
        // handle debugging events from the application...
        case "step": {
                :
        }
                :
                :
    }
};

// Send command to the application
webSocket.send("step-over");
```
