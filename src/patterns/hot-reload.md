Hot Reloading
=============

{{#include ../links.md}}


```admonish info "Usage scenario"

* A system where scripts are used for behavioral control.

* All or parts of the control scripts need to be modified dynamically without re-initializing the
  host system.

* New scripts must be active as soon as possible after modifications are detected.
```

```admonish abstract "Key concepts"

* The Rhai [`Engine`] is _re-entrant_, meaning that it is decoupled from scripts.

* A new script only needs to be recompiled and the new [`AST`] replaces the old for new behaviors
  to be active.

* Surgically _patch_ scripts when only parts of the scripts are modified.
```


Implementation
--------------

### Embed scripting engine and script into system

Say, a system has a Rhai [`Engine`] plus a compiled script (in [`AST`] form), with the [`AST`] kept
with interior mutability...

```rust
// Main system object
struct System {
    engine: Engine,
    script: Rc<RefCell<AST>>,
      :
}

// Embed Rhai 'Engine' and control script
let engine = Engine::new();
let ast = engine.compile_file("config.rhai")?;

let mut system = System { engine, script: Rc::new(RefCell::new(ast)) };

// Handle events with script functions
system.on_event(|sys: &System, event: &str, data: Map| {
    let mut scope = Scope::new();

    // Call script function which is the same name as the event
    sys.engine.call_fn(&mut scope, sys.script.borrow(), event, (data,)).unwrap();

    result
});
```

### Hot reload entire script upon change

If the control scripts are small enough and changes are infrequent, it is much simpler just to
recompile the whole set of script and replace the original [`AST`] with the new one.

```rust
// Watch for script file change
system.watch(|sys: &System, file: &str| {
    // Compile the new script
    let ast = sys.engine.compile_file(file.into())?;

    // Hot reload - just replace the old script!
    *sys.script.borrow_mut() = ast;

    Ok(())
});
```

### Hot patch specific functions

If the control scripts are large and complicated, and if the system can detect changes to specific [functions],
it is also possible to _patch_ just the changed [functions].

```rust
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
    *sys.script.borrow_mut() += patch_ast;
});
```


```admonish tip.small "Tip: Multi-threaded considerations"

For a multi-threaded environments, replace `Rc` with `Arc`, `RefCell` with `RwLock` or `Mutex`, and
turn on the [`sync`] feature.
```
