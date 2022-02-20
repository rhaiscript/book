Disable Looping
===============

{{#include ../links.md}}

For certain scripts, especially those in embedded usage for straight calculations, or where Rhai
script [`AST`]'s are eventually transcribed into some other instruction set, looping may be
undesirable as it may not be supported by the application itself.

Rhai looping constructs include the [`while`], [`loop`], [`do`] and [`for`] statements.

Although it is possible to disable these keywords via
[`Engine::disable_symbol`][disable keywords and operators], it is simpler to disable all looping
via [`Engine::set_allow_looping`][options].

```rust,no_run
use rhai::Engine;

let mut engine = Engine::new();

// Disable looping
engine.set_allow_looping(false);

// The following all return parse errors.

engine.compile("while x == y { x += 1; }")?;

engine.compile(r#"loop { print("hello world!"); }"#)?;

engine.compile("do { x += 1; } until x > 10;")?;

engine.compile("for n in 0..10 { print(n); }")?;
```
