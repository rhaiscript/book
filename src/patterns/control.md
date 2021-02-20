Scriptable Control Layer Over Rust Backend
=========================================

{{#include ../links.md}}


Usage Scenario
--------------

* A system provides core functionalities, but no driving logic.

* The driving logic must be dynamic and hot-loadable.

* A script is used to drive the system and provide control intelligence.


Key Concepts
------------

* Expose a Control API.

* Leverage [function overloading] to simplify the API design.

* Since Rhai is _[sand-boxed]_, it cannot mutate the environment.  To perform external actions via an API, the actual system must be wrapped in a `RefCell` (or `RwLock`/`Mutex` for [`sync`]) and shared to the [`Engine`].


Implementation
--------------

There are two broad ways for Rhai to control an external system, both of which involve
wrapping the system in a shared, interior-mutated object.

This is one way which does not involve exposing the data structures of the external system,
but only through exposing an abstract API primarily made up of functions.

Use this when the API is relatively simple and clean, and the number of functions is small enough.

For a complex API involving lots of functions, or an API that has a clear object structure,
use the [Singleton Command Object]({{rootUrl}}/patterns/singleton.md) pattern instead.


### Functional API

Assume that a system provides the following functional API:

```rust,no_run
struct EnergizerBunny;

impl EnergizerBunny {
    pub fn new () -> Self { ... }
    pub fn go (&mut self) { ... }
    pub fn stop (&mut self) { ... }
    pub fn is_going (&self) { ... }
    pub fn get_speed (&self) -> i64 { ... }
    pub fn set_speed (&mut self, speed: i64) { ... }
}
```

### Wrap API in Shared Object

```rust,no_run
pub type SharedBunny = Rc<RefCell<EnergizerBunny>>;
```

Note: Use `Arc<Mutex<T>>` or `Arc<RwLock<T>>` when using the [`sync`] feature because the function
must then be `Send + Sync`.

```rust,no_run
let bunny: SharedBunny = Rc::new(RefCell::(EnergizerBunny::new()));
```

### Register Control API

The trick to building a Control API is to clone the shared API object and
move it into each function registration via a closure.

Therefore, it is not possible to use a [plugin module] to achieve this, and each function must
be registered one after another.

```rust,no_run
// Notice 'move' is used to move the shared API object into the closure.
let b = bunny.clone();
engine.register_fn("bunny_power", move |on: bool| {
    if on {
        if b.borrow().is_going() {
            println!("Still going...");
        } else {
            b.borrow_mut().go();
        }
    } else {
        if b.borrow().is_going() {
            b.borrow_mut().stop();
        } else {
            println!("Already out of battery!");
        }
    }
});

let b = bunny.clone();
engine.register_fn("bunny_is_going", move || b.borrow().is_going());

let b = bunny.clone();
engine.register_fn("bunny_get_speed", move ||
    if b.borrow().is_going() { b.borrow().get_speed() } else { 0 }
);

let b = bunny.clone();
engine.register_result_fn("bunny_set_speed", move |speed: i64|
    if speed <= 0 {
        return Err("Speed must be positive!".into());
    } else if speed > 100 {
        return Err("Bunny will be going too fast!".into());
    }

    if b.borrow().is_going() {
        b.borrow_mut().set_speed(speed)
    } else {
        return Err("Bunny is not yet going!".into());
    }

    Ok(Dynamic::UNIT)
);
```

### Use the API

```rust,no_run
if !bunny_is_going() { bunny_power(true); }

if bunny_get_speed() > 50 { bunny_set_speed(50); }
```


Caveat
------

Although this usage pattern appears a perfect fit for _game_ logic, avoid writing the
_entire game_ in Rhai.  Performance will not be acceptable.

Implement as much functionalities of the game engine in Rust as possible.
Rhai integrates well with Rust so this is usually not a hinderance.

Lift as much out of Rhai as possible.
Use Rhai only for the logic that _must_ be dynamic or hot-loadable.
