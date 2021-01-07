Extend Rhai with Custom Syntax
=============================

{{#include ../links.md}}


For the ultimate adventurous, there is a built-in facility to _extend_ the Rhai language
with custom-defined _syntax_.

But before going off to define the next weird statement type, heed this warning:


Don't Do It™
------------

Stick with standard language syntax as much as possible.

Having to learn Rhai is bad enough, no sane user would ever want to learn _yet_ another
obscure language syntax just to do something.

Try to use [custom operators] first.  Defining a custom syntax should be considered a _last resort_.


Where This Might Be Useful
-------------------------

* Where an operation is used a _LOT_ and a custom syntax saves a lot of typing.

* Where a custom syntax _significantly_ simplifies the code and _significantly_ enhances understanding of the code's intent.

* Where certain logic cannot be easily encapsulated inside a function.

* Where you just want to confuse your user and make their lives miserable, because you can.


Step One &ndash; Design The Syntax
---------------------------------

A custom syntax is simply a list of symbols.

These symbol types can be used:

* Standard [keywords]({{rootUrl}}/appendix/keywords.md)

* Standard [operators]({{rootUrl}}/appendix/operators.md#operators).

* Reserved [symbols]({{rootUrl}}/appendix/operators.md#symbols).

* Identifiers following the [variable] naming rules.

* `$expr$` &ndash; any valid expression, statement or statement block.

* `$block$` &ndash; any valid statement block (i.e. must be enclosed by `'{'` .. `'}'`).

* `$ident$` &ndash; any [variable] name.

### The First Symbol Must be an Identifier

There is no specific limit on the combination and sequencing of each symbol type,
except the _first_ symbol which must be a custom keyword that follows the naming rules
of [variables].

The first symbol also cannot be a normal or reserved [keyword].
In other words, any valid identifier that is not a [keyword] will work fine.

### The First Symbol Must be Unique

Rhai uses the _first_ symbol as a clue to parse custom syntax.

Therefore, at any one time, there can only be _one_ custom syntax starting with each unique symbol.

Any new custom syntax definition using the same first symbol simply _overwrites_ the previous one.

### Example

```rust
exec $ident$ <- $expr$ : $block$
```

The above syntax is made up of a stream of symbols:

| Position | Input |  Symbol   | Description                                                                                              |
| :------: | :---: | :-------: | -------------------------------------------------------------------------------------------------------- |
|    1     |       |  `exec`   | custom keyword                                                                                           |
|    2     |   1   | `$ident$` | a variable name                                                                                          |
|    3     |       |   `<-`    | the left-arrow symbol (which is a [reserved symbol]({{rootUrl}}/appendix/operators.md#symbols) in Rhai). |
|    4     |   2   | `$expr$`  | an expression, which may be enclosed with `{` .. `}`, or not.                                            |
|    5     |       |    `:`    | the colon symbol                                                                                         |
|    6     |   3   | `$block$` | a statement block, which must be enclosed with `{` .. `}`.                                               |

This syntax matches the following sample code and generates three inputs (one for each non-keyword):

```rust
// Assuming the 'exec' custom syntax implementation declares the variable 'hello':
let x = exec hello <- foo(1, 2) : {
            hello += bar(hello);
            baz(hello);
        };

print(x);       // variable 'x'  has a value returned by the custom syntax

print(hello);   // variable declared by a custom syntax persists!
```


Step Two &ndash; Implementation
------------------------------

Any custom syntax must include an _implementation_ of it.

### Function Signature

The function signature of an implementation is:

> `Fn(context: &mut EvalContext, inputs: &[Expression]) -> Result<Dynamic, Box<EvalAltResult>>`

where:

| Parameter                  |                  Type                   | Description                                                                                                            |
| -------------------------- | :-------------------------------------: | ---------------------------------------------------------------------------------------------------------------------- |
| `context`                  |           `&mut EvalContext`            | mutable reference to the current evaluation _context_                                                                  |
| &bull; `scope()`           |                `&Scope`                 | reference to the current [`Scope`]                                                                                     |
| &bull; `scope_mut()`       |            `&mut &mut Scope`            | mutable reference to the current [`Scope`]; variables can be added to/removed from it                                  |
| &bull; `engine()`          |                `&Engine`                | reference to the current [`Engine`]                                                                                    |
| &bull; `source()`          |             `Option<&str>`              | reference to the current source, if any                                                                                |
| &bull; `iter_imports()`    | `impl Iterator<Item = (&str, &Module)>` | iterator of the current stack of [modules] imported via `import` statements                                            |
| &bull; `imports()`         |               `&Imports`                | reference to the current stack of [modules] imported via `import` statements; requires the [`internals`] feature       |
| &bull; `iter_namespaces()` |     `impl Iterator<Item = &Module>`     | iterator of the namespaces (as [modules]) containing all script-defined functions                                      |
| &bull; `namespaces()`      |              `&[&Module]`               | reference to the namespaces (as [modules]) containing all script-defined functions; requires the [`internals`] feature |
| &bull; `this_ptr()`        |           `Option<&Dynamic>`            | reference to the current bound [`this`] pointer, if any                                                                |
| &bull; `call_level()`      |                 `usize`                 | the current nesting level of function calls                                                                            |
| `inputs`                   |             `&[Expression]`             | a list of input expression trees                                                                                       |

### Return Value

Return value is the result of evaluating the custom syntax expression.

### Access Arguments

The most important argument is `inputs` where the matched identifiers (`$ident$`), expressions/statements (`$expr$`)
and statement blocks (`$block$`) are provided.

To access a particular argument, use the following patterns:

| Argument type | Pattern (`n` = slot in `inputs`)         | Result type  | Description        |
| :-----------: | ---------------------------------------- | :----------: | ------------------ |
|   `$ident$`   | `inputs[n].get_variable_name().unwrap()` |    `&str`    | name of a variable |
|   `$expr$`    | `inputs.get(n).unwrap()`                 | `Expression` | an expression tree |
|   `$block$`   | `inputs.get(n).unwrap()`                 | `Expression` | an expression tree |

### Evaluate an Expression Tree

Use the `EvalContext::eval_expression_tree` method to evaluate an arbitrary expression tree
within the current evaluation context.

```rust
let expression = inputs.get(0).unwrap();
let result = context.eval_expression_tree(expression)?;
```

### Declare Variables

New variables maybe declared (usually with a variable name that is passed in via `$ident$).

It can simply be pushed into the [`Scope`].

However, beware that all new variables must be declared _prior_ to evaluating any expression tree.
In other words, any [`Scope`] calls that change the list of must come _before_ any
`EvalContext::eval_expression_tree` calls.

```rust
let var_name = inputs[0].get_variable_name().unwrap();
let expression = inputs.get(1).unwrap();

context.scope_mut().push(var_name, 0 as INT);   // do this BEFORE 'context.eval_expression_tree'!

let result = context.eval_expression_tree(expression)?;
```


Step Three &ndash; Register the Custom Syntax
--------------------------------------------

Use `Engine::register_custom_syntax` to register a custom syntax.

Again, beware that the _first_ symbol must be unique.  If there already exists a custom syntax starting
with that symbol, the previous syntax will be overwritten.

The syntax is passed simply as a slice of `&str`.

```rust
// Custom syntax implementation
fn implementation_func(
    context: &mut EvalContext,
    inputs: &[Expression]
) -> Result<Dynamic, Box<EvalAltResult>> {
    let var_name = inputs[0].get_variable_name().unwrap().to_string();
    let stmt = inputs.get(1).unwrap();
    let condition = inputs.get(2).unwrap();

    // Push one new variable into the scope BEFORE 'context.eval_expression_tree'
    context.scope_mut().push(var_name, 0 as INT);

    loop {
        // Evaluate the statement block
        context.eval_expression_tree(stmt)?;

        // Evaluate the condition expression
        let stop = !context.eval_expression_tree(condition)?
                            .as_bool().map_err(|err| Box::new(
                                EvalAltResult::ErrorMismatchDataType(
                                    "bool".to_string(),
                                    err.to_string(),
                                    condition.position(),
                                )
                            ))?;

        if stop {
            break;
        }
    }

    Ok(Dynamic::UNIT)
}

// Register the custom syntax (sample): exec |x| -> { x += 1 } while x < 0
engine.register_custom_syntax(
    &[ "exec", "|", "$ident$", "|", "->", "$block$", "while", "$expr$" ], // the custom syntax
    1,  // the number of new variables declared within this custom syntax
    implementation_func
)?;
```

Remember that a custom syntax acts as an _expression_, so it can show up practically anywhere:

```rust
// Use as an expression:
let foo = (exec |x| -> { x += 1 } while x < 0) * 100;

// Use as a function call argument:
do_something(exec |x| -> { x += 1 } while x < 0, 24, true);

// Use as a statement:
exec |x| -> { x += 1 } while x < 0;
//                                ^ terminate statement with ';'
```


Step Four &ndash; Disable Unneeded Statement Types
-------------------------------------------------

When a DSL needs a custom syntax, most likely than not it is extremely specialized.
Therefore, many statement types actually may not make sense under the same usage scenario.

So, while at it, better [disable][disable keywords and operators] those built-in keywords
and operators that should not be used by the user.  The would leave only the bare minimum
language surface exposed, together with the custom syntax that is tailor-designed for
the scenario.

A keyword or operator that is disabled can still be used in a custom syntax.

In an extreme case, it is possible to disable _every_ keyword in the language, leaving only
custom syntax (plus possibly expressions).  But again, Don't Do It™ &ndash; unless you are certain
of what you're doing.


Step Five &ndash; Document
-------------------------

For custom syntax, documentation is crucial.

Make sure there are _lots_ of examples for users to follow.


Step Six &ndash; Profit!
------------------------


Really Advanced &ndash; Custom Parsers
-------------------------------------

Sometimes it is desirable to have multiple custom syntax starting with the
same symbol.  This is especially common for _command-style_ syntax where the
second symbol calls a particular command:

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

Alternatively, a custom syntax may have variable length, with a termination symbol:

```rust
// The following is a variable-length list terminated by '>'  
tags < "foo", "bar", 123, ... , x+y, true >
```

For even more flexibility in order to handle these advanced use cases, there is a
_low level_ API for custom syntax that allows the registration of an entire mini-parser.

Use `Engine::register_custom_syntax_raw` to register a custom syntax _parser_
together with the implementation function.

### How Custom Parsers Work

A custom parser takes as input parameters two pieces of information:

* The symbols parsed so far; `$ident$` is replaced with the actual identifier parsed,
  while `$expr$` and `$block$` stay as they were.
  
  The custom parser can inspect this symbols stream to determine the next symbol to parse.

* The _look-ahead_ symbol, which is the symbol that will be parsed _next_.

  If the look-ahead is an expected symbol, the customer parser just returns it to continue parsing,
  or it can return `$ident$` to parse it as an identifier, or even `$expr$` to start parsing
  an expression.

  If the look-ahead is '`{`', then the custom parser may also return `$block$` to start parsing a
  statements block.

  If the look-ahead is unexpected, the custom parser should then return the symbol expected
  and Rhai will fail with a parse error containing information about the expected symbol.

A custom parser always returns the _next_ symbol expected, which can also be `$ident$`,
`$expr$` or `$block$`, or `None` if parsing should terminate (_without_ reading the
look-ahead symbol).


### Example

```rust
engine.register_custom_syntax_raw(
    "perform",
    // The custom parser implementation - always returns the next symbol expected
    // 'look_ahead' is the next symbol about to be read
    |symbols, look_ahead| match symbols.len() {
        // perform ...
        1 => Ok(Some("$ident$".to_string())),
        // perform command ...
        2 => match symbols[1].as_str() {
            "action" => Ok(Some("$expr$".into())),
            "hello" => Ok(Some("world".into())),
            "update" | "check" | "add" | "remove" => Ok(Some("$ident$".into())),
            "cleanup" => Ok(None),
            cmd => Err(ParseError(Box::new(ParseErrorType::BadInput(
                LexError::ImproperSymbol(format!("Improper command: {}", cmd))
            )), Position::NONE)),
        },
        // perform command arg ...
        3 => match (symbols[1].as_str(), symbols[2].as_str()) {
            ("action", _) => Ok(None),
            ("hello", "world") => Ok(None),
            ("update", arg) if arg == "system" => Ok(None),
            ("update", arg) if arg == "client" => Ok(None),
            ("check", arg) => Ok(None),
            ("add", arg) => Ok(None),
            ("remove", arg) => Ok(None),
            (cmd, arg) => Err(ParseError(Box::new(ParseErrorType::BadInput(
                LexError::ImproperSymbol(
                    format!("Invalid argument for command {}: {}", cmd, arg)
                )
            )), Position::NONE)),
        },
        _ => unreachable!(),
    },
    // Number of new variables declared by this custom syntax
    0,
    // Implementation function
    implementation_func
);
```

### Function Signature

The custom syntax parser has the following signature:

> `Fn(symbols: &[ImmutableString], look_ahead: &str) -> Result<Option<ImmutableString>, ParseError>`

where:

| Parameter    |         Type         | Description                                                                                                                                    |
| ------------ | :------------------: | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| `symbols`    | `&[ImmutableString]` | a slice of symbols that have been parsed so far, possibly containing `$expr$` and/or `$block$`; `$ident$` is replaced by the actual identifier |
| `look_ahead` |        `&str`        | a string slice containing the next symbol that is about to be read                                                                             |

Most strings are [`ImmutableString`][string]'s so it is usually more efficient to just `clone` the appropriate one
(if any matches, or keep an internal cache for commonly-used symbols) as the return value.

### Return Value

The return value is `Result<Option<ImmutableString>, ParseError>` where:

| Value              | Description                                                                                                                                                                                                                         |
| ------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Ok(None)`         | parsing complete and there are no more symbols to match                                                                                                                                                                             |
| `Ok(Some(symbol))` | the next symbol to match, which can also be `$expr$`, `$ident$` or `$block$`                                                                                                                                                        |
| `Err(ParseError)`  | error that is reflected back to the [`Engine`] &ndash; normally `ParseError(ParseErrorType::BadInput(LexError::ImproperSymbol(message)), Position::NONE)` to indicate that there is a syntax error, but it can be any `ParseError`. |
