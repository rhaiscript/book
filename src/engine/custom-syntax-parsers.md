Really Advanced &ndash; Custom Parsers
======================================

{{#include ../links.md}}

Sometimes it is desirable to have multiple [custom syntax] starting with the same symbol.

This is especially common for _command-style_ syntax where the second symbol calls a particular command:

```rust
// The following simulates a command-style syntax, all starting with 'perform'.
perform hello world;        // A fixed sequence of symbols
perform action 42;          // Perform a system action with a parameter
perform update system;      // Update the system
perform check all;          // Check all system settings
perform cleanup;            // Clean up the system
perform add something;      // Add something to the system
perform remove something;   // Delete something from the system
```

Alternatively, a [custom syntax] may have variable length, with a termination symbol:

```rust
// The following is a variable-length list terminated by '>'  
tags < "foo", "bar", 123, ... , x+y, true >
```

For even more flexibility in order to handle these advanced use cases, there is a
_low level_ API for [custom syntax] that allows the registration of an entire mini-parser.

Use `Engine::register_custom_syntax_with_state_raw` to register a [custom syntax] _parser_ together
with an implementation function, both of which accept a custom user-defined _state_ value.


How Custom Parsers Work
-----------------------

### Leading Symbol

Under this API, the leading symbol for a custom parser is no longer restricted to be valid identifiers.

It can either be:

* an identifier that isn't a normal [keyword] unless [disabled][disable keywords and operators], or

* a valid symbol (see [list]({{rootUrl}}/appendix/operators.md)) which is not a normal [operator] unless [disabled][disable keywords and operators].

### Parser Function Signature

The [custom syntax] parser has the following signature.

> ```rust
> Fn(symbols: &[ImmutableString], look_ahead: &str, state: &mut Dynamic) -> Result<Option<ImmutableString>, ParseError>
> ```

where:

| Parameter    |                   Type                    | Description                                                                                                                                                         |
| ------------ | :---------------------------------------: | ------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `symbols`    | [`&[ImmutableString]`][`ImmutableString`] | a slice of symbols that have been parsed so far, possibly containing `$expr$` and/or `$block$`; `$ident$` and other literal markers are replaced by the actual text |
| `look_ahead` |                  `&str`                   | a string slice containing the next symbol that is about to be read                                                                                                  |
| `state`      |        [`&mut Dynamic`][`Dynamic`]        | mutable reference to a user-defined _state_                                                                                                                         |

Most strings are [`ImmutableString`]'s so it is usually more efficient to just `clone` the appropriate one
(if any matches, or keep an internal cache for commonly-used symbols) as the return value.

### Parameter #1 &ndash; Symbols Parsed So Far

The symbols parsed so far are provided as a slice of [`ImmutableString`]s.

The custom parser can inspect this symbols stream to determine the next symbol to parse.

| Argument type | Value             |
| :-----------: | ----------------- |
| text [string] | text value        |
|   `$ident$`   | identifier name   |
|  `$symbol$`   | symbol literal    |
|   `$expr$`    | `$expr$`          |
|   `$block$`   | `$block$`         |
|   `$bool$`    | `true` or `false` |
|    `$int$`    | value of number   |
|   `$float$`   | value of number   |
|  `$string$`   | [string] text     |

### Parameter #2 &ndash; Look-Ahead Symbol

The _look-ahead_ symbol is the symbol that will be parsed _next_.

If the look-ahead is an expected symbol, the customer parser just returns it to continue parsing,
or it can return `$ident$` to parse it as an identifier, or even `$expr$` to start parsing
an expression.

```admonish tip.side.wide "Tip: Strings vs identifiers"

The look-ahead of an identifier (e.g. [variable] name) is its text name.

That of a [string] literal is its content wrapped in _quotes_ (`"`), e.g. `"this is a string"`.
```

If the look-ahead is `{`, then the custom parser may also return `$block$` to start parsing a
statements block.

If the look-ahead is unexpected, the custom parser should then return the symbol expected
and Rhai will fail with a parse error containing information about the expected symbol.

### Parameter #3 &ndash; User-Defined Custom _State_

The _state's_ value starts off as [`()`].

Its type is [`Dynamic`], possible to hold any value.

Usually it is set to an [object map] that contains information on the state of parsing.

### Return value

The return value is `Result<Option<ImmutableString>, ParseError>` where:

|       Value        | Description                                                                                                                                                                                                                              |
| :----------------: | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
|     `Ok(None)`     | parsing is complete and there is no more symbol to match                                                                                                                                                                                 |
| `Ok(Some(symbol))` | the next `symbol` to match, which can also be `$expr$`, `$ident$`, `$block$` etc.                                                                                                                                                        |
|    `Err(error)`    | `error` that is reflected back to the [`Engine`] &ndash; normally `ParseError( ParseErrorType::BadInput( LexError::ImproperSymbol(message) ), Position::NONE)` to indicate that there is a syntax error, but it can be any `ParseError`. |

A custom parser always returns `Some` with the _next_ symbol expected (which can be `$ident$`,
`$expr$`, `$block$` etc.) or `None` if parsing should terminate (_without_ reading the
look-ahead symbol).

#### The `$$` return symbol short-cut

A return symbol starting with `$$` is treated specially.

Like `None`, it also terminates parsing, but at the same time it adds this symbol as text into the
_inputs_ stream at the end.

This is typically used to inform the implementation function which [custom syntax] variant was
actually parsed.

```rust
fn implementation_fn(context: &mut EvalContext, inputs: &[Expression], state: &Dynamic) -> Result<Dynamic, Box<EvalAltResult>>
{
    // Get the last symbol
    let key = inputs.last().unwrap().get_string_value().unwrap();

    // Make sure it starts with '$$'
    assert!(key.starts_with("$$"));

    // Execute the custom syntax expression
    match key {
        "$$hello" => { ... }
        "$$world" => { ... }
        "$$foo" => { ... }
        "$$bar" => { ... }
        _ => Err(...)
    }
}
```

`$$` is a convenient _short-cut_. An alternative method is to pass such information in the user-defined
custom _state_.

### Implementation Function Signature

The signature of an implementation function for `Engine::register_custom_syntax_with_state_raw` is
as follows, which is slightly different from the function for `Engine::register_custom_syntax`.

> ```rust
> Fn(context: &mut EvalContext, inputs: &[Expression], state: &Dynamic) -> Result<Dynamic, Box<EvalAltResult>>
> ```

where:

| Parameter |                Type                 | Description                                           |
| --------- | :---------------------------------: | ----------------------------------------------------- |
| `context` | [`&mut EvalContext`][`EvalContext`] | mutable reference to the current _evaluation context_ |
| `inputs`  |           `&[Expression]`           | a list of input expression trees                      |
| `state`   |       [`&Dynamic`][`Dynamic`]       | reference to the user-defined state                   |


Custom Parser Example
---------------------

```rust
engine.register_custom_syntax_with_state_raw(
    // The leading symbol - which needs not be an identifier.
    "perform",
    // The custom parser implementation - always returns the next symbol expected
    // 'look_ahead' is the next symbol about to be read
    //
    // Return symbols starting with '$$' also terminate parsing but allows us
    // to determine which syntax variant was actually parsed so we can perform the
    // appropriate action.  This is a convenient short-cut to keeping the value
    // inside the state.
    //
    // The return type is 'Option<ImmutableString>' to allow common text strings
    // to be interned and shared easily, reducing allocations during parsing.
    |symbols, look_ahead, state| match symbols.len() {
        // perform ...
        1 => Ok(Some("$ident$".into())),
        // perform command ...
        2 => match symbols[1].as_str() {
            "action" => Ok(Some("$expr$".into())),
            "hello" => Ok(Some("world".into())),
            "update" | "check" | "add" | "remove" => Ok(Some("$ident$".into())),
            "cleanup" => Ok(Some("$$cleanup".into())),
            cmd => Err(LexError::ImproperSymbol(format!("Improper command: {cmd}"))
                       .into_err(Position::NONE)),
        },
        // perform command arg ...
        3 => match (symbols[1].as_str(), symbols[2].as_str()) {
            ("action", _) => Ok(Some("$$action".into())),
            ("hello", "world") => Ok(Some("$$hello-world".into())),
            ("update", arg) => match arg {
                "system" => Ok(Some("$$update-system".into())),
                "client" => Ok(Some("$$update-client".into())),
                _ => Err(LexError::ImproperSymbol(format!("Cannot update {arg}"))
                         .into_err(Position::NONE))
            },
            ("check", arg) => Ok(Some("$$check".into())),
            ("add", arg) => Ok(Some("$$add".into())),
            ("remove", arg) => Ok(Some("$$remove".into())),
            (cmd, arg) => Err(LexError::ImproperSymbol(
                format!("Invalid argument for command {cmd}: {arg}")
            ).into_err(Position::NONE)),
        },
        _ => unreachable!(),
    },
    // No variables declared/removed by this custom syntax
    false,
    // Implementation function
    |context, inputs, state| {
        let cmd = inputs.last().unwrap().get_string_value().unwrap();

        match cmd {
            "$$cleanup" => { ... }
            "$$action" => { ... }
            "$$update-system" => { ... }
            "$$update-client" => { ... }
            "$$check" => { ... }
            "$$add" => { ... }
            "$$remove" => { ... }
            _ => Err(format!("Invalid command: {cmd}"))
        }
    }
);
```
