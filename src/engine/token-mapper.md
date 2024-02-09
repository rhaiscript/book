Remap Tokens During Parsing
===========================

{{#include ../links.md}}

[`Token`]: https://docs.rs/rhai/{{version}}/rhai/enum.Token.html


The Rhai [`Engine`] first parses a script into a stream of _tokens_.

Tokens have the type [`Token`] which is only exported under [`internals`].

The function `Engine::on_parse_token`, available only under [`internals`], allows registration of a
_mapper function_ that converts (remaps) a [`Token`] into another.


```admonish tip.small "Hot Tips: Use as safety checks"

Since it is called for _every_ token parsed from the script, this token mapper function
can also be used to implement _safety checks_ against, say, stack-overflow or out-of-memory
situations during parsing.

See [here][memory] for more details.
```


Function Signature
------------------

```admonish tip.side "Tip: Raising errors"

Raise a parse error by returning [`Token::LexError`](https://docs.rs/rhai/{{version}}/rhai/enum.Token.html#variant.LexError)
as the mapped token.
```

The function signature passed to `Engine::on_parse_token` takes the following form.

> ```rust
> Fn(token: Token, pos: Position, state: &TokenizeState) -> Token
> ```

where:

| Parameter |                                        Type                                         | Description                      |
| --------- | :---------------------------------------------------------------------------------: | -------------------------------- |
| `token`   |                                      [`Token`]                                      | the next symbol parsed           |
| `pos`     |                                     `Position`                                      | location of the [token][`Token`] |
| `state`   | [`&TokenizeState`](https://docs.rs/rhai/{{version}}/rhai/struct.TokenizeState.html) | current state of the tokenizer   |


Example
-------

```rust
use rhai::{Engine, FLOAT, Token};

let mut engine = Engine::new();

// Register a token mapper function.
engine.on_parse_token(|token, pos, state| {
    match token {
        // Change 'begin' ... 'end' to '{' ... '}'
        Token::Identifier(s) if &s == "begin" => Token::LeftBrace,
        Token::Identifier(s) if &s == "end" => Token::RightBrace,

        // Change all integer literals to floating-point
        Token::IntegerConstant(n) => Token::FloatConstant((n as FLOAT).into()),
        
        // Disallow '()'
        Token::Unit => Token::LexError(
            LexError::ImproperSymbol("()".to_string(), "".to_string()).into()
        ),

        // Pass through all other tokens unchanged
        _ => token
    }
});
```
