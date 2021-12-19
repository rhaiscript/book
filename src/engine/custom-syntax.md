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


### Step One &ndash; Design The Syntax

A custom syntax is simply a list of symbols.

These symbol types can be used:

* Standard [keywords]({{rootUrl}}/appendix/keywords.md)
* Standard [operators]({{rootUrl}}/appendix/operators.md#operators).
* Reserved [symbols]({{rootUrl}}/appendix/operators.md#symbols).
* Identifiers following the [variable] naming rules.
* `$expr$` &ndash; any valid expression, statement or statement block.
* `$block$` &ndash; any valid statement block (i.e. must be enclosed by `{` .. `}`).
* `$ident$` &ndash; any [variable] name.
* `$symbol$` &ndash; any [symbol]({{rootUrl}}/appendix/operators.md}}), active or reserved.
* `$bool$` &ndash; a boolean value.
* `$int$` &ndash; an integer number.
* `$float$` &ndash; a floating-point number (if not [`no_float`]).
* `$string$` &ndash; a [string] literal.

#### The first symbol must be an identifier

There is no specific limit on the combination and sequencing of each symbol type,
except the _first_ symbol which must be a custom keyword that follows the naming rules
of [variables].

The first symbol also cannot be a normal [keyword] unless it is [disabled][disable keywords and operators].
Any valid identifier that is not an active [keyword] works fine, even if it is a reserved [keyword].

#### The first symbol must be unique

Rhai uses the _first_ symbol as a clue to parse custom syntax.

Therefore, at any one time, there can only be _one_ custom syntax starting with each unique symbol.

Any new custom syntax definition using the same first symbol simply _overwrites_ the previous one.

#### Example

```rust no_run
exec [ $ident$ $symbol$ $int$ ] <- $expr$ : $block$
```

The above syntax is made up of a stream of symbols:

| Position | Input slot |   Symbol   | Description                                                                                              |
| :------: | :--------: | :--------: | -------------------------------------------------------------------------------------------------------- |
|    1     |            |   `exec`   | custom keyword                                                                                           |
|    2     |            |    `[`     | the left bracket symbol                                                                                  |
|    2     |     0      | `$ident$`  | a variable name                                                                                          |
|    3     |     1      | `$symbol$` | the operator                                                                                             |
|    4     |     2      |  `$int$`   | an integer number                                                                                        |
|    5     |            |    `]`     | the right bracket symbol                                                                                 |
|    6     |            |    `<-`    | the left-arrow symbol (which is a [reserved symbol]({{rootUrl}}/appendix/operators.md#symbols) in Rhai). |
|    7     |     3      |  `$expr$`  | an expression, which may be enclosed with `{` .. `}`, or not.                                            |
|    8     |            |    `:`     | the colon symbol                                                                                         |
|    9     |     4      | `$block$`  | a statement block, which must be enclosed with `{` .. `}`.                                               |

This syntax matches the following sample code and generates three inputs (one for each non-keyword):

```rust no_run
// Assuming the 'exec' custom syntax implementation declares the variable 'hello':
let x = exec [hello < 42] <- foo(1, 2) : {
            hello += bar(hello);
            baz(hello);
        };

print(x);       // variable 'x'  has a value returned by the custom syntax

print(hello);   // variable declared by a custom syntax persists!
```


### Step Two &ndash; Implementation

Any custom syntax must include an _implementation_ of it.

#### Function signature

The function signature of an implementation is:

> `Fn(context: &mut EvalContext, inputs: &[Expression]) -> Result<Dynamic, Box<EvalAltResult>>`

where:

| Parameter |        Type        | Description                                           |
| --------- | :----------------: | ----------------------------------------------------- |
| `context` | `&mut EvalContext` | mutable reference to the current _evaluation context_ |
| `inputs`  |  `&[Expression]`   | a list of input expression trees                      |

and `EvalContext` is a type that encapsulates the current _evaluation context_ and exposes the following:

| Method              |               Return type               | Description                                                                                                                                     |
| ------------------- | :-------------------------------------: | ----------------------------------------------------------------------------------------------------------------------------------------------- |
| `scope()`           |                `&Scope`                 | reference to the current [`Scope`]                                                                                                              |
| `scope_mut()`       |            `&mut &mut Scope`            | mutable reference to the current [`Scope`]; variables can be added to/removed from it                                                           |
| `engine()`          |                `&Engine`                | reference to the current [`Engine`]                                                                                                             |
| `source()`          |             `Option<&str>`              | reference to the current source, if any                                                                                                         |
| `iter_imports()`    | `impl Iterator<Item = (&str, &Module)>` | iterator of the current stack of [modules] imported via `import` statements                                                                     |
| `imports()`         |               `&Imports`                | reference to the current stack of [modules] imported via `import` statements; requires the [`internals`] feature                                |
| `iter_namespaces()` |     `impl Iterator<Item = &Module>`     | iterator of the [namespaces][function namespaces] (as [modules]) containing all script-defined [functions]                                      |
| `namespaces()`      |              `&[&Module]`               | reference to the [namespaces][function namespaces] (as [modules]) containing all script-defined [functions]; requires the [`internals`] feature |
| `this_ptr()`        |           `Option<&Dynamic>`            | reference to the current bound [`this`] pointer, if any                                                                                         |
| `call_level()`      |                 `usize`                 | the current nesting level of function calls                                                                                                     |


#### Return value

Return value is the result of evaluating the custom syntax expression.

#### Access arguments

The most important argument is `inputs` where the matched identifiers (`$ident$`), expressions/statements (`$expr$`)
and statement blocks (`$block$`) are provided.

To access a particular argument, use the following patterns:

| Argument type | Pattern (`n` = slot in `inputs`)                                                                             |             Result type             | Description           |
| :-----------: | ------------------------------------------------------------------------------------------------------------ | :---------------------------------: | --------------------- |
|   `$ident$`   | `inputs[n].get_string_value().unwrap()`                                                                      |               `&str`                | variable name         |
|  `$symbol$`   | `inputs[n].get_literal_value::<ImmutableString>().unwrap()`                                                  |         [`ImmutableString`]         | symbol literal        |
|   `$expr$`    | `&inputs[n]`                                                                                                 |            `&Expression`            | an expression tree    |
|   `$block$`   | `&inputs[n]`                                                                                                 |            `&Expression`            | an expression tree    |
|   `$bool$`    | `inputs[n].get_literal_value::<bool>().unwrap()`                                                             |               `bool`                | boolean value         |
|    `$int$`    | `inputs[n].get_literal_value::<INT>().unwrap()`                                                              |                `INT`                | integer number        |
|   `$float$`   | `inputs[n].get_literal_value::<FLOAT>().unwrap()`                                                            |               `FLOAT`               | floating-point number |
|  `$string$`   | `inputs[n].get_literal_value::<ImmutableString>().unwrap()`<br/><br/>`inputs[n].get_string_value().unwrap()` | [`ImmutableString`]<br/><br/>`&str` | [string] text         |

#### Get literal constants

Several argument types represent literal constants that can be obtained directly via
`Expression::get_literal_value<T>` or `Expression::get_string_value` (for [strings]).

```rust no_run
let expression = &inputs[0];

// Use 'get_literal_value' with a turbo-fish type to extract the value
let string_value = expression.get_literal_value::<ImmutableString>().unwrap();
let string_slice = expression.get_string_value().unwrap();

let float_value = expression.get_literal_value::<FLOAT>().unwrap();

// Or assign directly to a variable with type...
let int_value: INT = expression.get_literal_value().unwrap();

// Or use type inference!
let bool_value = expression.get_literal_value().unwrap();

if bool_value { ... }       // 'bool_value' inferred to be 'bool'
```

#### Evaluate an expression tree

Use the `EvalContext::eval_expression_tree` method to evaluate an arbitrary expression tree
within the current evaluation context.

```rust no_run
let expression = &inputs[0];
let result = context.eval_expression_tree(expression)?;
```

#### Declare variables

New variables maybe declared (usually with a variable name that is passed in via `$ident$`).

It can simply be pushed into the [`Scope`].

However, beware that all new variables must be declared _prior_ to evaluating any expression tree.
In other words, any [`Scope`] calls that change the list of must come _before_ any
`EvalContext::eval_expression_tree` calls.

```rust no_run
let var_name = inputs[0].get_string_value().unwrap();
let expression = &inputs[1];

context.scope_mut().push(var_name, 0_i64);      // do this BEFORE 'context.eval_expression_tree'!

let result = context.eval_expression_tree(expression)?;
```


### Step Three &ndash; Register the Custom Syntax

Use `Engine::register_custom_syntax` to register a custom syntax.

Again, beware that the _first_ symbol must be unique.  If there already exists a custom syntax starting
with that symbol, the previous syntax will be overwritten.

The syntax is passed simply as a slice of `&str`.

```rust no_run
// Custom syntax implementation
fn implementation_func(context: &mut EvalContext, inputs: &[Expression]) -> Result<Dynamic, Box<EvalAltResult>> {
    let var_name = inputs[0].get_string_value().unwrap();
    let stmt = &inputs[1];
    let condition = &inputs[2];

    // Push new variable into the scope BEFORE 'context.eval_expression_tree'
    context.scope_mut().push(var_name.to_string(), 0_i64);

    let mut count = 0_i64;

    loop {
        // Evaluate the statement block
        context.eval_expression_tree(stmt)?;

        count += 1;

        // Declare a new variable every three turns...
        if count % 3 == 0 {
            context.scope_mut().push(format!("{}{}", var_name, count), count);
        }

        // Evaluate the condition expression
        let expr_result = !context.eval_expression_tree(condition)?;

        match expr_result.as_bool() {
            Ok(true) => (),
            Ok(false) => break,
            Err(err) => return Err(EvalAltResult::ErrorMismatchDataType(
                            "bool".to_string(),
                            err.to_string(),
                            condition.position(),
                        ).into()),
        }
    }

    Ok(Dynamic::UNIT)
}

// Register the custom syntax (sample): exec<x> -> { x += 1 } while x < 0
engine.register_custom_syntax(
    &[ "exec", "<", "$ident$", ">", "->", "$block$", "while", "$expr$" ], // the custom syntax
    true,  // variables declared within this custom syntax
    implementation_func
)?;
```

Remember that a custom syntax acts as an _expression_, so it can show up practically anywhere:

```rust no_run
// Use as an expression:
let foo = (exec<x> -> { x += 1 } while x < 42) * 100;

// New variables are successfully declared...
x == 42;
x3 == 3;
x6 == 6;

// Use as a function call argument:
do_something(exec<x> -> { x += 1 } while x < 42, 24, true);

// Use as a statement:
exec<x> -> { x += 1 } while x < 0;
//                               ^ terminate statement with ';' unless the custom
//                                 syntax already ends with '}'
```


### Step Four &ndash; Disable Unneeded Statement Types

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


### Step Five &ndash; Document

For custom syntax, documentation is crucial.

Make sure there are _lots_ of examples for users to follow.


### Step Six &ndash; Profit!


Practical Example &ndash; Recreating JavaScript's `var` Statement
---------------------------------------------------------------

The following example recreates a statement similar to the `var` variable declaration syntax in
JavaScript, which creates a global variable if one doesn't already exist.
There is currently no equivalent in Rhai.

```rust no_run
// Register the custom syntax: var x = ???
engine.register_custom_syntax(&[ "var", "$ident$", "=", "$expr$" ], true, |context, inputs| {
    let var_name = inputs[0].get_string_value().unwrap().to_string();
    let expr = &inputs[1];

    // Evaluate the expression
    let value = context.eval_expression_tree(expr)?;

    // Push a new variable into the scope if it doesn't already exist.
    // Otherwise just set its value.
    if !context.scope().is_constant(var_name).unwrap_or(false) {
        context.scope_mut().set_value(var_name.to_string(), value);
        Ok(Dynamic::UNIT)
    } else {
        Err(format!("variable {} is constant", var_name).into())
    }
})?;
```


Really Advanced &ndash; Custom Parsers
-------------------------------------

Sometimes it is desirable to have multiple custom syntax starting with the same symbol.
This is especially common for _command-style_ syntax where the second symbol calls a particular command:

```rust no_run
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

```rust no_run
// The following is a variable-length list terminated by '>'  
tags < "foo", "bar", 123, ... , x+y, true >
```

For even more flexibility in order to handle these advanced use cases, there is a
_low level_ API for custom syntax that allows the registration of an entire mini-parser.

Use `Engine::register_custom_syntax_raw` to register a custom syntax _parser_
together with the implementation function.


How Custom Parsers Work
-----------------------

### Leading symbol

The leading symbol for a custom parser can either be:

* a identifier that isn't a normal [keyword] unless [disabled][disable keywords and operators], or

* a valid symbol (see [list]({{rootUrl}}/appendix/operators.md)) which is not a normal operator unless [disabled][disable keywords and operators].

Under this API, it is no longer restricted to be valid identifiers.

### Function Signature

The custom syntax parser has the following signature:

> `Fn(symbols: &[ImmutableString], look_ahead: &str) -> Result<Option<ImmutableString>, ParseError>`

where:

| Parameter    |         Type         | Description                                                                                                                                                         |
| ------------ | :------------------: | ------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `symbols`    | `&[ImmutableString]` | a slice of symbols that have been parsed so far, possibly containing `$expr$` and/or `$block$`; `$ident$` and other literal markers are replaced by the actual text |
| `look_ahead` |        `&str`        | a string slice containing the next symbol that is about to be read                                                                                                  |

Most strings are [`ImmutableString`]'s so it is usually more efficient to just `clone` the appropriate one
(if any matches, or keep an internal cache for commonly-used symbols) as the return value.

### Parameters

A custom parser takes as input parameters two pieces of information:

* The symbols (as [`ImmutableString`]s) parsed so far:
  
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

  The custom parser can inspect this symbols stream to determine the next symbol to parse.

* The _look-ahead_ symbol, which is the symbol that will be parsed _next_.

  If the look-ahead is an expected symbol, the customer parser just returns it to continue parsing,
  or it can return `$ident$` to parse it as an identifier, or even `$expr$` to start parsing
  an expression.

  If the look-ahead is `{`, then the custom parser may also return `$block$` to start parsing a
  statements block.

  If the look-ahead is unexpected, the custom parser should then return the symbol expected
  and Rhai will fail with a parse error containing information about the expected symbol.

### Return value

The return value is `Result<Option<ImmutableString>, ParseError>` where:

| Value              | Description                                                                                                                                                                                                                         |
| ------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Ok(None)`         | parsing complete and there are no more symbols to match                                                                                                                                                                             |
| `Ok(Some(symbol))` | the next symbol to match, which can also be `$expr$`, `$ident$` or `$block$`                                                                                                                                                        |
| `Err(ParseError)`  | error that is reflected back to the [`Engine`] &ndash; normally `ParseError(ParseErrorType::BadInput(LexError::ImproperSymbol(message)), Position::NONE)` to indicate that there is a syntax error, but it can be any `ParseError`. |

A custom parser always returns `Some` with the _next_ symbol expected (which can be `$ident$`,
`$expr$`, `$block$` etc.) or `None` if parsing should terminate (_without_ reading the
look-ahead symbol).

A return symbol starting with `$$` is treated specially. Like returning `None`, it also
terminates parsing, but at the same time it adds this symbol as text into the _inputs_ stream at the end.
This is typically used to inform the implementation function which custom syntax variant was
actually parsed.


### Example

```rust no_run
engine.register_custom_syntax_raw(
    // The leading symbol - which needs not be an identifier.
    "perform",
    // The custom parser implementation - always returns the next symbol expected
    // 'look_ahead' is the next symbol about to be read
    //
    // Return symbols starting with '$$' terminate parsing but also allows us
    // to determine which syntax variant was actually parsed so we can perform the
    // appropriate action.
    //
    // The return type is 'Option<ImmutableString>' to allow common text strings
    // to be interned and shared easily, reducing allocations during parsing.
    |symbols, look_ahead| match symbols.len() {
        // perform ...
        1 => Ok(Some("$ident$".into())),
        // perform command ...
        2 => match symbols[1].as_str() {
            "action" => Ok(Some("$expr$".into())),
            "hello" => Ok(Some("world".into())),
            "update" | "check" | "add" | "remove" => Ok(Some("$ident$".into())),
            "cleanup" => Ok(Some("$$cleanup".into())),
            cmd => Err(ParseError(Box::new(ParseErrorType::BadInput(
                LexError::ImproperSymbol(format!("Improper command: {}", cmd))
            )), Position::NONE)),
        },
        // perform command arg ...
        3 => match (symbols[1].as_str(), symbols[2].as_str()) {
            ("action", _) => Ok(Some("$$action".into())),
            ("hello", "world") => Ok(Some("$$hello-world".into())),
            ("update", arg) if arg == "system" => Ok(Some("$$update-system".into())),
            ("update", arg) if arg == "client" => Ok(Some("$$update-client".into())),
            ("check", arg) => Ok(Some("$$check".into())),
            ("add", arg) => Ok(Some("$$add".into())),
            ("remove", arg) => Ok(Some("$$remove".into())),
            (cmd, arg) => Err(ParseError(Box::new(ParseErrorType::BadInput(
                LexError::ImproperSymbol(
                    format!("Invalid argument for command {}: {}", cmd, arg)
                )
            )), Position::NONE)),
        },
        _ => unreachable!(),
    },
    // No variables declared/removed by this custom syntax
    false,
    // Implementation function
    implementation_func
);
```
