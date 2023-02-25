Compile a Script (to AST)
=========================

{{#include ../links.md}}

To repeatedly evaluate a script, _compile_ it first with `Engine::compile` into an `AST`
(**A**bstract **S**yntax **T**ree) form.

`Engine::eval_ast_XXX` and `Engine::run_ast_XXX` evaluate a pre-compiled `AST`.

```rust
// Compile to an AST and store it for later evaluations
let ast = engine.compile("40 + 2")?;

for _ in 0..42 {
    let result: i64 = engine.eval_ast(&ast)?;

    println!("Answer #{i}: {result}");      // prints 42
}
```

~~~admonish tip.small "Tip: Compile script file"

Compiling script files is also supported via `Engine::compile_file`
(not available for [`no_std`] or [WASM] builds).

```rust
let ast = engine.compile_file("hello_world.rhai".into())?;
```
~~~

~~~admonish info.small "See also: `AST` manipulation API"

Advanced users who may want to manipulate an `AST`, especially the functions contained within,
should see the section on [_Manage AST's_](ast.md) for more details.
~~~


Practical Use &ndash; Header Template Scripts
---------------------------------------------

Sometimes it is desirable to include a standardized _header template_ in a script that contains
pre-defined [functions], [constants] and [imported][`import`] [modules].

```rust
// START OF THE HEADER TEMPLATE
// The following should run before every script...

import "hello" as h;
import "world" as w;

// Standard constants

const GLOBAL_CONSTANT = 42;
const SCALE_FACTOR = 1.2;

// Standard functions

fn foo(x, y) { ... }

fn bar() { ... }

fn baz() { ... }

// END OF THE HEADER TEMPLATE

// Everything below changes from run to run

foo(bar() + GLOBAL_CONSTANT, baz() * SCALE_FACTOR)
```

### Option 1 &ndash; The easy way

Prepend the script header template onto independent scripts and run them as a whole.

> **Pros:** Easy!  
>
> **Cons:** If the header template is long, work is duplicated every time to parse it.

```rust
let header_template = "..... // scripts... .....";

for index in 0..10000 {
    let user_script = db.get_script(index);

    // Just merge the two scripts...
    let combined_script = format!("{header_template}\n{user_script}\n");

    // Run away!
    let result = engine.eval::<i64>(combined_script)?;
    
    println!("{result}");
}
```

### Option 2 &ndash; The hard way

Option 1 requires the script header template to be recompiled every time.  This can be expensive if
the header is very long.

This option compiles both the script header template and independent scripts as separate `AST`'s
which are then joined together to form a combined `AST`.

> **Pros:** No need to recompile the header template!  
>
> **Cons:** More work...

```rust
let header_template = "..... // scripts... .....";

let mut template_ast = engine.compile(header_template)?;

// If you don't want to run the template, only keep the functions
// defined inside (e.g. closures), clear out the statements.
template_ast.clear_statements();

for index in 0..10000 {
    let user_script = db.get_script(index);

    let user_ast = engine.compile(user_script)?;

    // Merge the two AST's
    let combined_ast = template_ast + user_ast;

    // Run away!
    let result = engine.eval_ast::<i64>(combined_ast)?;
    
    println!("{result}");
```

### Option 3 &ndash; The not-so-hard way

Option 1 does repeated work, option 2 requires manipulating `AST`'s...

This option makes the scripted [functions] (not [imported][`import`] [modules] nor [constants]
however) available globally by first making it a [module] (via [`Module::eval_ast_as_new`](modules/ast.md))
and then loading it into the [`Engine`] via `Engine::register_global_module`.

> **Pros:** No need to recompile the header template!  
>
> **Cons:** No [imported][`import`] [modules] nor [constants]; if the header template is changed, a new [`Engine`] must be created.

```rust
let header_template = "..... // scripts... .....";

let template_ast = engine.compile(header_template)?;

let template_module = Module::eval_ast_as_new(Scope::new(), &template_ast, &engine)?;

engine.register_global_module(template_module.into());

for index in 0..10000 {
    let user_script = db.get_script(index);

    // Run away!
    let result = engine.eval::<i64>(user_script)?;
    
    println!("{result}");
}
```
