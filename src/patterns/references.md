Passing External References to Rhai (Unsafe)
============================================

{{#include ../links.md}}


[`transmute`]: https://doc.rust-lang.org/std/mem/fn.transmute.html
[`Engine::set_default_tag`]: https://docs.rs/rhai/{{version}}/rhai/struct.Engine.html#method.set_default_tag


```admonish danger "Don't Do It™"

As with anything `unsafe`, don't do this unless you have exhausted all possible alternatives.
There are usually alternatives.
```

```admonish info "Usage scenario"

* A system where an embedded [`Engine`] is used to call scripts within a callback function/closure
  from some external system.  This is extremely common when working with an ECS (Entity Component System).

* Said external system provides only _references_ (mutable or immutable) to work with their internal states.

* Such states are not available outside of references (and therefore cannot be made shared).

* Said external system's API require such states to function.

* It is impossible to pass references into Rhai because the [`Engine`] does not track lifetimes.
```

```admonish abstract "Key concepts"

* Make a newtype that wraps an integer which is set to the CPU's pointer size (typically `usize`).

* [`transmute`] the reference provided by the system into an integer and store it within the newtype.
  This requires `unsafe` code.

* Use the newtype as a _handle_ for all registered API functions, [`transmute`] the integer back
  to a reference before use. This also requires `unsafe` code.

* Make absolutely sure that the newtype is never stored anywhere permanent (e.g. in a [`Scope`])
  not does it ever live outside of the reference's scope.
```


Example
-------

```rust
/// Newtype wrapping a reference (pointer) cast into 'usize'
/// together with a unique ID for protection.
#[derive(Debug, Eq, PartialEq, Clone, Hash)]
pub struct WorldHandle(usize, i64);

/// Create handle from reference
impl From<&mut World> for WorldHandle {
    fn from(world: &mut World) -> Self {
        Self::new(world)
    }
}

/// Recover reference from handle
impl AsMut<World> for WorldHandle {
    fn as_mut(&mut self) -> &mut World {
        unsafe { std::mem::transmute(self.0) }
    }
}

impl WorldHandle {
    /// Create handle from reference, using a random number as unique ID
    pub fn new(world: &mut World) -> Self {
        let handle =unsafe { std::mem::transmute(world) };
        let unique_id = rand::random();

        Self(handle, unique_id)
    }

    /// Get the unique ID of this instance
    pub fn unique_id(&self) -> i64 {
        self.1
    }
}

/// API for handle to 'World'
#[export_module]
pub mod handle_module {
    pub type Handle = WorldHandle;

    /// Draw a bunch of pretty shapes.
    #[rhai_fn(return_raw)]
    pub fn draw(context: NativeCallContext, handle: &mut Handle, shapes: Array) -> Result<(), Box<EvalAltResult>> {
        // Double check the pointer is still fresh
        // by comparing the handle's unique ID with
        // the version stored in the engine's tag!
        if handle.unique_id() != context.tag() {
            return "Ouch! The handle is stale!".into();
        }

        // Get the reference to 'World'
        let world: &mut World = handle.as_mut();

        // ... work with reference
        world.draw_really_pretty_shapes(shapes);

        Ok(())
    }
}

// Register API for handle type
engine.register_global_module(exported_module!(handle_module).into());

// Successfully pass a reference to 'World' into Rhai
super_ecs_system.query(...).for_each(|world: &mut World| {
    // Create handle from reference to 'World'
    let handle: WorldHandle = world.into();

    // Set the handle's ID into the engine's tag
    engine.set_default_tag(handle.unique_id());

    // Alternatively, use 'Engine::call_fn_raw_raw' and set the handle's ID
    // into the 'tag' field of the 'GlobalRuntimeState' object

    // Add handle into scope
    let mut scope = Scope::new();
    scope.push("world", handle);
    
    // Work with handle
    engine.run_with_scope(&mut scope, "world.draw([1, 2, 3])")

    // Make sure no instance of 'handle' leaks outside this scope!
});
```


```admonish danger "IMPORTANT! Ensure no leakage!"

It is of utmost importance that no instance of the handle newtype ever _leaks_ outside of the
appropriate scope where the wrapped reference may no longer be valid.

For example, do not allow the script to store a copy of the handle anywhere that can
potentially be persistent (e.g. within an [object map]).

#### Safety check via unique ID

One solution, as illustrated in the example, is to always tag each handle instance together with
a unique random ID. That same ID can then be set into the [`Engine`] (via [`Engine::set_default_tag`])
before running scripts.

Before executing any API function, first check whether the handle's ID matches that of the current [`Engine`]
(via [`NativeCallContext::tag`][`NativeCallContext`]). If they differ, the handle is stale and should never be used;
an error should be returned instead.

#### Alternative to `Engine::set_default_tag`

Alternatively, if the [`Engine`] cannot be made mutable, use `Engine::call_fn_raw_raw`
(which requires the [`internals`] feature) to directly call a script [function] in a compiled [`AST`].

This volatile API allows specifying a [`GlobalRuntimeState`] object which contains the `tag` field
that can be set just for that evaluation run.
```
