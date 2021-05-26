Singleton Command Object
=======================

{{#include ../links.md}}


Usage Scenario
--------------

* A system provides core functionalities, but no driving logic.

* The driving logic must be dynamic and hot-loadable.

* A script is used to drive the system and provide control intelligence.

* The API is multiplexed, meaning that it can act on multiple system-provided entities, or

* The API lends itself readily to an object-oriented (OO) representation.


Key Concepts
------------

* Expose a Command type with an API.  The [`no_object`] feature must not be on.

* Leverage [function overloading] to simplify the API design.

* Since Rhai is _[sand-boxed]_, it cannot mutate the environment.  To perform external actions via an API, the command object type must be wrapped in a `RefCell` (or `RwLock`/`Mutex` for [`sync`]) and shared to the [`Engine`].

* Load each command object into a custom [`Scope`] as constant variables.

* Control each command object in script via the constants.


Implementation
--------------

There are two broad ways for Rhai to control an external system, both of which involve
wrapping the system in a shared, interior-mutated object.

This is the other way which involves directly exposing the data structures of the external system
as a name singleton object in the scripting space.

Use this when the API is complex but has a clear object structure.

For a relatively simple API that is action-based and not object-based,
use the [Control Layer]({{rootUrl}}/patterns/control.md) pattern instead.


### Functional API

Assume the following command object type:

```rust , no_run
struct EnergizerBunny { ... }

impl EnergizerBunny {
    pub fn new () -> Self { ... }
    pub fn go (&mut self) { ... }
    pub fn stop (&mut self) { ... }
    pub fn is_going (&self) -> bol { ... }
    pub fn get_speed (&self) -> i64 { ... }
    pub fn set_speed (&mut self, speed: i64) { ... }
    pub fn turn (&mut self, left_turn: bool) { ... }
}
```

### Wrap Command Object Type as Shared

```rust , no_run
pub type SharedBunny = Rc<RefCell<EnergizerBunny>>;
```

or in multi-threaded environments with the [`sync`] feature, use one of the following:

```rust , no_run
pub type SharedBunny = Arc<RwLock<EnergizerBunny>>;

pub type SharedBunny = Arc<Mutex<EnergizerBunny>>;
```

### Register the Custom Type

```rust , no_run
engine.register_type_with_name::<SharedBunny>("EnergizerBunny");
```

### Develop a Plugin with Methods and Getters/Setters

The easiest way to develop a complete set of API for a [custom type] is via a [plugin module].

Notice that putting `pure` in `#[rhai_fn(...)]` allows a [getter/setter][getters/setters] to operate
on a [constant] without raising an error.

```rust , no_run
use rhai::plugin::*;

#[export_module]
pub mod bunny_api {
    pub const MAX_SPEED: i64 = 100;

    #[rhai_fn(get = "power", pure)]
    pub fn get_power(bunny: &mut SharedBunny) -> bool {
        bunny.borrow().is_going()
    }
    #[rhai_fn(set = "power", pure)]
    pub fn set_power(bunny: &mut SharedBunny, on: bool) {
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
    pub fn get_speed(bunny: &mut SharedBunny) -> i64 {
        if bunny.borrow().is_going() {
            bunny.borrow().get_speed()
        } else {
            0
        }
    }
    #[rhai_fn(set = "speed", pure, return_raw)]
    pub fn set_speed(bunny: &mut SharedBunny, speed: i64)
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
    pub fn turn_left(bunny: &mut SharedBunny) {
        if bunny.borrow().is_going() {
            bunny.borrow_mut().turn(true);
        }
    }
    pub fn turn_right(bunny: &mut SharedBunny) {
        if bunny.borrow().is_going() {
            bunny.borrow_mut().turn(false);
        }
    }
}

engine.register_global_module(exported_module!(bunny_api).into());
```

### Compile script into AST

```rust , no_run
let ast = engine.compile(script)?;
```

### Push Constant Command Object into Custom Scope and Run AST

```rust , no_run
let bunny: SharedBunny = Rc::new(RefCell::new(EnergizerBunny::new()));

let mut scope = Scope::new();

// Add the command object into a custom Scope.
// Constants, as a convention, are named with all-capital letters.
scope.push_constant("BUNNY", bunny.clone());

// Run the compiled AST
engine.consume_ast_with_scope(&mut scope, &ast)?;

// Running the script directly, as below, is less desirable because
// the constant 'BUNNY' will be propagated and copied into each usage
// during the script optimization step
engine.consume_with_scope(&mut scope, script)?;
```

### Use the Command API in Script

```rust , no_run
// Access the command object via constant variable 'BUNNY'.

if !BUNNY.power { BUNNY.power = true; }

if BUNNY.speed > 50 { BUNNY.speed = 50; }

BUNNY.turn_left();
```
