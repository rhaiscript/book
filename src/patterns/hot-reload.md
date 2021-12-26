Hot Reloading
=============

{{#include ../links.md}}


Usage Scenario
--------------

* A system where scripts are used for behavioral control.

* All or parts of the control scripts need to be modified dynamically without re-initializing the host system.

* New scripts must be active as soon as possible after modifications are detected.


Key Concepts
------------

* The Rhai [`Engine`] is _re-entrant_, meaning that it is decoupled from scripts.

* A new script only needs to be recompiled and the new [`AST`] replaces the old for new behaviors to be active.

* Surgically _patch_ scripts when only parts of the scripts are modified.


Implementation
--------------

### Embed scripting engine and script into system

Say, a system has a Rhai [`Engine`] plus a compiled script (in [`AST`] form)...

```rust no_run
// Main system object
struct System {
    engine: Engine,
    script: AST,
      :
}

let mut system = System::new();

// Embed Rhai 'Engine' and control script
system.engine = Engine::new();
system.ast = system.engine.compile_file("config.rhai")?;

// Handle events with script functions
system.on_event(|sys: &System, event: &str, data: Map| {
    let mut scope = Scope::new();

    // Call script function which is the same name as the event
    sys.engine.call_fn(&mut scope, &sys.ast, event, (data,)).unwrap();

    result
});
```

### Hot reload entire script upon change

If the control scripts are small enough and changes are infrequent, it is much simpler just to
recompile the whole set of script and replace the original [`AST`] with the new one.

```rust no_run
// Watch for script file change
system.watch(|sys: &mut System, file: &str| {
    // Compile the new script
    let ast = sys.engine.compile_file(file.into())?;

    // Hot reload - just replace the old script!
    sys.ast = ast;
});
```

### Hot load specific functions via patching

If the control scripts are large and complicated, and if the system can detect changes to specific [functions],
it is also possible to _patch_ just the changed [functions].

```rust no_run
// Watch for changes in the script
system.watch_for_script_change(|sys: &mut System, fn_name: &str| {
    // Get the script file that contains the function
    let script = get_script_file_path(fn_name);

    // Compile the new script
    let mut patch_ast = sys.engine.compile_file(script)?;

    // Remove everything other than the specified function
    patch_ast.clear_statements();
    patch_ast.retain_functions(|_, _, name, _| name == fn_name);

    // Hot reload (via +=) only those functions in the script!
    sys.ast += patch_ast;
});
```
