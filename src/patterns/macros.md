Simulate Macros to Simplify Scripts
===================================

{{#include ../links.md}}


```admonish info "Usage scenario"

* Scripts need to access existing data in [variables].

* The particular fields to access correspond to long/complex expressions (e.g. long
  [indexing][indexers] and/or [property][getters/setters] chains `foo[x][y].bar[z].baz`).

* Usage is prevalent inside the scripts, requiring extensive duplications of code that
  are prone to typos and errors.

* There are a few such [variables] to modify at the same time &ndash; otherwise, it would
  be simpler to bind the `this` pointer to the [variable].
```

```admonish abstract "Key concepts"

* Pick a _macro_ syntax that is unlikely to conflict with content in literal [strings].

* Before script evaluation/compilation, globally replace macros with their corresponding expansions.
```


Pick a Macro Syntax
-------------------

```admonish danger.side "Warning: Not real macros"

The technique described here is to _simulate_ macros.
They are not _REAL_ macros.
```

Pick a syntax that is intuitive for the domain but **unlikely to occur naturally inside
[string] literals**.

| Sample Syntax | Sample usage       |
| ------------- | ------------------ |
| `#FOO`        | `#FOO = 42;`       |
| `$Bar`        | `$Bar.work();`     |
| `<Baz>`       | `print(<Baz>);`    |
| `#HELLO#`     | `let x = #HELLO#;` |
| `%HEY%`       | `%HEY% += 1;`      |

~~~admonish tip.small "Tip: Avoid normal words"

Avoid normal syntax that may show up inside a [string] literal.

For example, if using `Target` as a macro:

```rust
// This script...
Target.do_damage(10);

if Target.hp <= 0 {
    print("Target is destroyed!");
}

// Will turn to this...
entities["monster"].do_damage(10);

if entities["monster"].hp <= 0 {
    // Text in string literal erroneously replaced!
    print("entities["monster"] is destroyed!");
}
```
~~~


Global Search/Replace
---------------------

```rust
// Replace macros with expansions
let script = script.replace("#FOO", "foo[x][y].bar[z].baz");

let mut scope = Scope::new();

// Add global variables
scope.push("foo", ...);
scope.push_constant("x", ...);
scope.push_constant("y", ...);
scope.push_constant("z", ...);

// Run the script as normal
engine.run_with_scope(&mut scope, script)?;
```

~~~admonish example.small "Example script"

```js
print(`Found entity FOO at (${x},${y},${z})`);

let speed = #FOO.speed;

if speed < 42 {
    #FOO.speed *= 2;
} else {
    #FOO.teleport(#FOO.home());
}

print(`FOO is now at (${ #FOO.current_location() })`);
```
~~~

```admonish bug.small "Character positions"

After macro expansion, the _character positions_ of different script
elements will be shifted based on the length of the expanded text.

Therefore, error positions may no longer point to the correct locations
in the original, unexpanded scripts.

Line numbers are not affected.
```
