Switch Expression
=================

{{#include ../links.md}}

Like [`if`], [`switch`] also works as an _expression_.

```admonish tip.small "Tip"

This means that a [`switch`] expression can appear anywhere a regular expression can,
e.g. as [function] call arguments.
```

~~~admonish tip.small "Tip: Disable `switch` expressions"

[`switch`] expressions can be disabled via [`Engine::set_allow_switch_expression`][options].
~~~

```js
let x = switch foo { 1 => true, _ => false };

func(switch foo {
    "hello" => 42,
    "world" => 123,
    _ => 0
});

// The above is somewhat equivalent to:

let x = if foo == 1 { true } else { false };

if foo == "hello" {
    func(42);
} else if foo == "world" {
    func(123);
} else {
    func(0);
}
```
