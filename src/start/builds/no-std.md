`no-std` Build
=============

{{#include ../../links.md}}

The feature [`no_std`] automatically converts the scripting engine into a `no-std` build.

Usually, a `no-std` build goes hand-in-hand with [minimal builds] because typical embedded
hardware (the primary target for `no-std`) has limited storage.


Nightly Required
----------------

Currently, [`no_std`] requires the nightly compiler due to the crates that it uses.


Implementation
--------------

Rhai allocates, so the first thing that must be included in any `no-std` project is
an allocator crate, such as [`wee_alloc`](https://crates.io/crates/wee_alloc).

Then there is the need to set up proper error/panic handlers.
The following example uses `panic = "abort"` and `wee_alloc` as the allocator.

```rust
// Set up for no-std.
#![no_std]

// The following no-std features are usually needed.
#![feature(alloc_error_handler, start, core_intrinsics, lang_items, link_cfg)]

// Set up the global allocator.
extern crate alloc;
extern crate wee_alloc;

#[global_allocator]
static ALLOC: wee_alloc::WeeAlloc = wee_alloc::WeeAlloc::INIT;

// Rust needs a CRT runtime on Windows when compiled with MSVC.
#[cfg(all(windows, target_env = "msvc"))]
#[link(name = "msvcrt")]
#[link(name = "libcmt")]
extern {}

// Set up panic and error handlers
#[alloc_error_handler]
fn err_handler(_: core::alloc::Layout) -> ! {
    core::intrinsics::abort();
}

#[panic_handler]
#[lang = "panic_impl"]
extern "C" fn rust_begin_panic(_: &core::panic::PanicInfo) -> ! {
    core::intrinsics::abort();
}

#[lang = "eh_personality"]
extern "C" fn eh_personality() {}

#[no_mangle]
extern "C" fn rust_eh_register_frames() {}

#[no_mangle]
extern "C" fn rust_eh_unregister_frames() {}

#[start]
fn main(_argc: isize, _argv: *const *const u8) -> isize {
    // ... main program ...
}
```


Samples
-------

Check out the [`no-std` sample applications](../examples/rust.md#no-std-samples)
for different operating environments.
