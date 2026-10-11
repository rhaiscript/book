`no-std` Build
==============

{{#include ../../links.md}}


The [`no_std`] feature lets Rhai run in a Rust `no_std` application, primarily for embedded devices.

Since Rhai allocates, the application must provide an allocator, such as [`wee_alloc`](https://crates.io/crates/wee_alloc).

A `no-std` build is often combined with [minimal builds] because embedded hardware usually have
limited storage.

The sample below uses [`wee_alloc`](https://crates.io/crates/wee_alloc) as allocator and
evaluates a simple expression using a [raw `Engine`].


`Cargo.toml`
------------

```toml
[dependencies]
rhai = { version = "{{version}}", default-features = false, features = ["no_std"] }
wee_alloc = { version = "0.4.5", default-features = false }

[profile.dev]
panic = "abort"

[profile.release]
opt-level = "z"
debug = false
rpath = false
debug-assertions = false
codegen-units = 1
panic = "abort"

[profile.unix]
inherits = "release"
lto = true

[profile.windows]
inherits = "release"

[profile.macos]
inherits = "release"
lto = "fat"
```

Application
-----------

```rust
#![no_main]
#![no_std]
#![feature(alloc_error_handler, abort_immediate)]

extern crate alloc;
extern crate wee_alloc;

#[global_allocator]
static ALLOC: wee_alloc::WeeAlloc = wee_alloc::WeeAlloc::INIT;

#[cfg(all(windows, target_env = "msvc"))]
#[link(name = "msvcrt")]
#[link(name = "libcmt")]
extern "C" {}

use rhai::{Engine, INT};

#[no_mangle]
pub extern "C" fn main(_argc: isize, _argv: *const *const u8) -> isize {
    // A raw engine starts with only the built-in operators.
    // Load packages from `rhai::packages` to add more functionality.
    let engine = Engine::new_raw();

    engine.eval_expression::<INT>("40 + 2").unwrap() as isize
}

#[alloc_error_handler]
fn foo(_: core::alloc::Layout) -> ! {
    core::process::abort_immediate()
}

#[panic_handler]
fn rust_begin_panic(_: &core::panic::PanicInfo) -> ! {
    core::process::abort_immediate()
}

#[no_mangle]
extern "C" fn _rust_eh_personality() {}

#[no_mangle]
extern "C" fn rust_eh_personality() {}

#[no_mangle]
extern "C" fn rust_eh_register_frames() {}

#[no_mangle]
extern "C" fn rust_eh_unregister_frames() {}

#[no_mangle]
extern "C" fn _Unwind_Resume() {}
```

Building
--------

This example uses nightly-only Rust features for the allocator error handler and immediate abort.

Build it with the nightly toolchain:

```bash
cargo +nightly build --release
```

The sample defines `unix`, `windows`, and `macos` profiles, which inherit the size-optimized release
profile and can be selected by name:

```bash
cargo +nightly build --profile unix
```

The release profile is optimized for size; it can be changed to optimize for speed instead.

```admonish example.small "Sample"

Check out the [`no-std` sample applications]({{rootUrl}}/start/examples/rust.html#no-std-examples)
for different operating environments.
```
