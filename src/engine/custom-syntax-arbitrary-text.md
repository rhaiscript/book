Really Really Advanced &ndash; Parsing Arbitrary Text
=====================================================

{{#include ../links.md}}

Sometimes, in a DSL, it is desirable to have arbitrary text processed by a custom parser.

For instance, in order to parse the following embedded HTML custom syntax:

```html
let node = HTML
    <div>
        <h1>Hello, World!</h1>
        <p class="greeting">This is a simple HTML fragment.</p>
    </div>;
```

instead of the less user-friendly:

```js
let node = parse_html(`
    <div>
        <h1>Hello, World!</h1>
        <p class="greeting">This is a simple HTML fragment.</p>
    </div>
`);

```


The Power of `$raw$`
---------------------

Use `Engine::register_custom_syntax_without_look_ahead_raw` to register a [custom syntax] _parser_
that allows the use of `$raw$`. Look-ahead is not supported if `$raw$` is used.

`$raw$` simply returns the script text character-by-character without any processing (not even
whitespace or [comments]), by-passing Rhai's tokenizer completely.


Parser Function Signature
-------------------------

The [custom syntax] parser signature for `Engine::register_custom_syntax_without_look_ahead_raw` has
no `look_ahead` parameter.

> ```rust
> Fn(symbols: &[ImmutableString], state: &mut Dynamic) -> Result<Option<ImmutableString>, ParseError>
> ```

where:

| Parameter |                   Type                    | Description                                                                                                                                                                                                                       |
| --------- | :---------------------------------------: | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `symbols` | [`&[ImmutableString]`][`ImmutableString`] | a slice of symbols that have been parsed so far, possibly containing `$expr$` and/or `$block$`; `$ident$` and other literal markers are replaced by the actual text; `$raw$` is replaced by the next character in the script text |
| `state`   |        [`&mut Dynamic`][`Dynamic`]        | mutable reference to a user-defined _state_                                                                                                                                                                                       |

### Return value

The return value is `Result<Option<ImmutableString>, ParseError>` where:

|       Value        | Description                                                                                                                                                                                                                              |
| :----------------: | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
|     `Ok(None)`     | parsing is complete and there is no more symbol to match                                                                                                                                                                                 |
| `Ok(Some(symbol))` | the next `symbol` to match, which can also be `$expr$`, `$ident$`, `$block$` etc. or `$raw$`                                                                                                                                             |
|    `Err(error)`    | `error` that is reflected back to the [`Engine`] &ndash; normally `ParseError( ParseErrorType::BadInput( LexError::ImproperSymbol(message) ), Position::NONE)` to indicate that there is a syntax error, but it can be any `ParseError`. |

Return `Ok(Some("$raw$"))` to indicate that the next token should simply be the next character in
the script text. No processing is performed. Whitespaces and [comments] are passed verbatim.


Example
-------

```rust
// Common strings as clonable 'ImmutableString'
let raw_str: ImmutableString = "$raw$".into();
let inner_str: ImmutableString = "$inner$".into();
let ident_str: ImmutableString = "$ident$".into();

// This custom parser parses raw SQL text, replacing '{...}' blocks or '@xxx' variables
// with the corresponding values.
engine.register_custom_syntax_without_look_ahead_raw(
    "SELECT",
    move |symbols, state| {
        // Build a text string as the state
        let mut text: String = if state.is_unit() { Default::default() } else { state.take().cast::<ImmutableString>().into() };

        // At every iteration, the last symbol is the new one
        let r = match symbols.last().unwrap().as_str() {
            // Terminate parsing when we see `;`
            ";" => None,
            // Variable substitution -- parse the following as a block
            "{" => Some(inner_str.clone()), // return '$inner$'
            // Block parsed, replace it with `?` as parameter
            "$inner$" => {
                text.push('?');
                Some(raw_str.clone())   // return '$raw$'
            }
            // Variable substitution -- parse the following as an identifier
            "@" => {
                text.push('@');
                Some(ident_str.clone()) // return '$ident$'
            }
            // Variable parsed, replace it with `?` as parameter
            _ if text.ends_with('@') => {
                let _ = text.pop().unwrap();
                text.push('?');
                Some(raw_str.clone())   // return '$raw$'
            }
            // Otherwise simply concat the tokens
            s => {
                text.push_str(s);
                Some(raw_str.clone())   // return '$raw$'
            }
        };

        // SQL statement done!
        *state = text.into();

        Ok(r)
    },
    false,
    ...
);
```

The above custom parser can parse the following script. Notice that whitespaces and even `//` that
normally starts a [comment] are preserved.

```sql
let nobody = "John Doe";
let min_amount = 100.0;
let max_total = 1000.0;

let records = SELECT
                id,
                SUM(amount) AS `total`,
                FIRST('http://hitme.com/') + id AS `link`
              FROM db.public.users
              WHERE name <> {nobody} AND amount >= {min_amount}
              GROUP BY id
              HAVING SUM(amount) <= @max_total;
```

The state will contain the following prepared SQL statement:

```sql
SELECT
                id,
                SUM(amount) AS `total`,
                FIRST('http://hitme.com/') + id AS `link`
              FROM db.public.users
              WHERE name <> ? AND amount >= ?
              GROUP BY id
              HAVING SUM(amount) <= ?
```

with the following parameters as inputs:

```rust
@1 = "John Doe"
@2 = 100.0
@3 = 1000.0
```

that the implementation function can take to query for data from a backend database.
