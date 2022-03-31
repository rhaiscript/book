Singleton Command Object
========================

{{#include ../links.md}}


```admonish info "Usage scenario"

* A system provides core functionalities, but no driving logic.

* The driving logic must be dynamic and hot-loadable.

* A script is used to drive the system and provide control intelligence.

* The API is multiplexed, meaning that it can act on multiple system-provided entities, or

* The API lends itself readily to an object-oriented (OO) representation.
```

```admonish abstract "Key concepts"

* Expose a Command type with an API.  The [`no_object`] feature must not be on.

* Leverage [function overloading] to simplify the API design.

* Since Rhai is _[sand-boxed]_, it cannot mutate the environment.  To perform external actions via
  an API, the command object type must be wrapped in a `RefCell` (or `RwLock`/`Mutex` for [`sync`])
  and shared to the [`Engine`].

* Load each command object into a custom [`Scope`] as constant variables.

* Control each command object in script via the constants.
```


Implementation
--------------

There are two broad ways for Rhai to control an external system, both of which involve wrapping the
system in a shared, interior-mutated object.

This is the other way which involves directly exposing the data structures of the external system as
a name singleton object in the scripting space.

Use this when the API is complex but has a clear object structure.

For a relatively simple API that is action-based and not object-based, use the
[Control Layer]({{rootUrl}}/patterns/control.md) pattern instead.


### Functional API

Assume the following command object type:

```rust
struct EnergizerBunny { ... }

impl EnergizerBunny {
    pub fn new () -> Self { ... }
    pub fn go (&mut self) { ... }
    pub fn stop (&mut self) { ... }
    pub fn is_going (&self) -> bool { ... }
    pub fn get_speed (&self) -> i64 { ... }
    pub fn set_speed (&mut self, speed: i64) { ... }
    pub fn turn (&mut self, left_turn: bool) { ... }
}
```

### Wrap command object type as shared

```rust
pub type SharedBunny = Rc<RefCell<EnergizerBunny>>;
```

or in multi-threaded environments with the [`sync`] feature, use one of the following:

```rust
pub type SharedBunny = Arc<RwLock<EnergizerBunny>>;

pub type SharedBunny = Arc<Mutex<EnergizerBunny>>;
```

### Develop a plugin with methods and getters/setters

The easiest way to develop a complete set of API for a [custom type] is via a [plugin module].

Notice that putting `pure` in `#[rhai_fn(...)]` allows a [getter/setter][getters/setters] to operate
on a [constant] without raising an error.  Therefore, it is needed on _all_ functions.

```rust
use rhai::plugin::*;

// Remember to put 'pure' on all functions, or they'll choke on constants!

#[export_module]
pub mod bunny_api {
    // Custom type 'SharedBunny' will be called 'EnergizerBunny' in scripts
    pub type EnergizerBunny = SharedBunny;

    // This constant is also available to scripts
    pub const MAX_SPEED: i64 = 100;

    #[rhai_fn(get = "power", pure)]
    pub fn get_power(bunny: &mut EnergizerBunny) -> bool {
        bunny.borrow().is_going()
    }
    #[rhai_fn(set = "power", pure)]
    pub fn set_power(bunny: &mut EnergizerBunny, on: bool) {
        if on {
            if bunny.borrow().is_going() {
                println!("Still going...");
            } else {
                bunny.borrow_mut().go();
            }
        } else {
            if bunny.borrow().is_going() {
                bunny.borrow_mut().stop();
            } else {
                println!("Already out of battery!");
            }
        }
    }
    #[rhai_fn(get = "speed", pure)]
    pub fn get_speed(bunny: &mut EnergizerBunny) -> i64 {
        if bunny.borrow().is_going() {
            bunny.borrow().get_speed()
        } else {
            0
        }
    }
    #[rhai_fn(set = "speed", pure, return_raw)]
    pub fn set_speed(bunny: &mut EnergizerBunny, speed: i64)
            -> Result<(), Box<EvalAltResult>>
    {
        if speed <= 0 {
            Err("Speed must be positive!".into())
        } else if speed > MAX_SPEED {
            Err("Bunny will be going too fast!".into())
        } else if !bunny.borrow().is_going() {
            Err("Bunny is not yet going!".into())
        } else {
            b.borrow_mut().set_speed(speed);
            Ok(())
        }
    }
    #[rhai_fn(pure)]
    pub fn turn_left(bunny: &mut EnergizerBunny) {
        if bunny.borrow().is_going() {
            bunny.borrow_mut().turn(true);
        }
    }
    #[rhai_fn(pure)]
    pub fn turn_right(bunny: &mut EnergizerBunny) {
        if bunny.borrow().is_going() {
            bunny.borrow_mut().turn(false);
        }
    }
}

engine.register_global_module(exported_module!(bunny_api).into());
```

```admonish tip.small "Tip: Friendly name for custom type"

It is customary to register a _friendly display name_ for any [custom type]
involved in the plugin.

This can easily be done via a _type alias_ in the plugin module.
```

### Compile script into AST

```rust
let ast = engine.compile(script)?;
```

### Push constant command object into custom scope and run AST

```rust
let bunny: SharedBunny = Rc::new(RefCell::new(EnergizerBunny::new()));

let mut scope = Scope::new();

// Add the singleton command object into a custom Scope.
// Constants, as a convention, are named with all-capital letters.
scope.push_constant("BUNNY", bunny.clone());

// Run the compiled AST
engine.run_ast_with_scope(&mut scope, &ast)?;

// Running the script directly, as below, is less desirable because
// the constant 'BUNNY' will be propagated and copied into each usage
// during the script optimization step
engine.run_with_scope(&mut scope, script)?;
```

~~~admonish tip.small "Tip: Prevent shadowing"

It is usually desirable to prevent [shadowing] of the singleton command object.

This can be easily achieved via a [variable definition filter].

```rust
// Now the script can no longer define a variable named 'BUNNY'
engine.on_def_var(|_, info, _| Ok(info.name != "BUNNY"));
```
~~~

### Use the command API in script

```rust
// Access the command object via constant variable 'BUNNY'.

if !BUNNY.power { BUNNY.power = true; }

if BUNNY.speed > 50 { BUNNY.speed = 50; }

BUNNY.turn_left();

let BUNNY = 42;     // <- syntax error if variable definition filter is set
```
