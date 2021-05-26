Implement a Custom Module Resolver
=================================

{{#include ../../links.md}}

For many applications in which Rhai is embedded, it is necessary to customize the way that modules
are resolved.  For instance, modules may need to be loaded from script texts stored in a database,
not in the file system.

A module resolver must implement the [`ModuleResolver`][traits] trait,
which contains only one required function: `resolve`.

When Rhai prepares to load a module, `ModuleResolver::resolve` is called with the name
of the _module path_ (i.e. the path specified in the [`import`] statement).

* Upon success, it should return an [`Rc<Module>`][module] (or [`Arc<Module>`][module] under [`sync`]).
  
  The module should call `Module::build_index` on the target module before returning.
  This method flattens the entire module tree and _indexes_ it for fast function name resolution.
  If the module is already indexed, calling this method has no effect.

* If the path does not resolve to a valid module, return `EvalAltResult::ErrorModuleNotFound`.

* If the module failed to load, return `EvalAltResult::ErrorInModule`.


Example of a Custom Module Resolver
----------------------------------

```rust , no_run
use rhai::{ModuleResolver, Module, Engine, EvalAltResult};

// Define a custom module resolver.
struct MyModuleResolver {}

// Implement the 'ModuleResolver' trait.
impl ModuleResolver for MyModuleResolver {
    // Only required function.
    fn resolve(
        &self,
        engine: &Engine,                    // reference to the current 'Engine'
        source_path: Option<&str>,          // path of the parent module
        path: &str,                         // the module path
        pos: Position,                      // position of the 'import' statement
    ) -> Result<Rc<Module>, Box<EvalAltResult>> {
        // Check module path.
        if is_valid_module_path(path) {
            let mut my_module =
                load_secret_module(path)    // load the custom module
                    .map_err(|err|
                        // Return EvalAltResult::ErrorInModule upon loading error
                        EvalAltResult::ErrorInModule(path.into(), Box::new(err), pos).into()
                    )?;
            my_module.build_index();        // index it
            Rc::new(my_module)              // make it shared
        } else {
            // Return EvalAltResult::ErrorModuleNotFound if the path is invalid
            Err(EvalAltResult::ErrorModuleNotFound(path.into(), pos).into())
        }
    }
}

let mut engine = Engine::new();

// Set the custom module resolver into the 'Engine'.
engine.set_module_resolver(MyModuleResolver {});

engine.consume(
r#"
    import "hello" as foo;  // this 'import' statement will call
                            // 'MyModuleResolver::resolve' with "hello" as 'path'
    foo:bar();
"#)?;
```


Implementing `ModuleResolver::resolve_ast`
-----------------------------------------

There is another function in the [`ModuleResolver`][traits] trait, `resolve_ast`, which is a
low-level API intended for advanced usage scenarios.

`ModuleResolver::resolve_ast` has a default implementation that simply returns `None`,
which indicates that this API is not supported by the [module resolver].

Any [module resolver] that serves [modules] based on Rhai scripts should implement
`ModuleResolver::resolve_ast`. When called, the compiled [`AST`] of the script should be returned.

`ModuleResolver::resolve_ast` should not return an error if `ModuleResolver::resolve` will not.
On the other hand, the same error should be returned if `ModuleResolver::resolve` will return one.
