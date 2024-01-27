Extend Rhai with Custom Syntax
==============================

{{#include ../links.md}}

For the ultimate adventurous, there is a built-in facility to _extend_ the Rhai language with
custom-defined _syntax_.

But before going off to define the next weird statement type, heed this warning:

```admonish danger.small "Don't Do It™"

Stick with standard language syntax as much as possible.

Having to learn Rhai is bad enough, no sane user would ever want to learn _yet_ another obscure
language syntax just to do something.

Try [custom operators] first.  A custom syntax should be considered a _last resort_.
```

```admonish success.small "Where this might be useful"

* Where an operation is used a _LOT_ and a custom syntax saves a lot of typing.

* Where a custom syntax _significantly_ simplifies the code and _significantly_ enhances
  understanding of the code's intent.

* Where certain logic cannot be easily encapsulated inside a function.

* Where you just want to confuse your user and make their lives miserable, because you can.
```

```admonish tip.small "Disable custom syntax"

Custom syntax can be disabled via the [`no_custom_syntax`] feature.
```


How to Do It
------------

### Step One &ndash; Design The Syntax

A custom syntax is simply a list of symbols.

These symbol types can be used:

* Standard [keywords]
* Standard [operators]
* Reserved [symbols]({{rootUrl}}/appendix/operators.md#symbols).
* Identifiers following the [variable] naming rules.
* `$expr$` &ndash; any valid expression, statement or statements block.
* `$block$` &ndash; any valid statements block (i.e. must be enclosed by `{` ... `}`).
* `$ident$` &ndash; any [variable] name.
* `$symbol$` &ndash; any [symbol][operator], active or reserved.
* `$bool$` &ndash; a boolean value.
* `$int$` &ndash; an integer number.
* `$float$` &ndash; a floating-point number (if not [`no_float`]).
* `$string$` &ndash; a [string] literal.

#### The first symbol must be an identifier

There is no specific limit on the combination and sequencing of each symbol type,
except the _first_ symbol which must be a custom [keyword] that follows the naming rules
of [variables].

The first symbol also cannot be a normal [keyword] unless it is [disabled][disable keywords and operators].
Any valid identifier that is not an active [keyword] works fine, even if it is a reserved [keyword].

#### The first symbol must be unique

Rhai uses the _first_ symbol as a clue to parse custom syntax.

Therefore, at any one time, there can only be _one_ custom syntax starting with each unique symbol.

Any new custom syntax definition using the same first symbol simply _overwrites_ the previous one.

#### Example

```rust
exec [ $ident$ $symbol$ $int$ ] <- $expr$ : $block$
```

The above syntax is made up of a stream of symbols:

| Position | Input slot |   Symbol   | Description                                                                                              |
| :------: | :--------: | :--------: | -------------------------------------------------------------------------------------------------------- |
|    1     |            |   `exec`   | custom keyword                                                                                           |
|    2     |            |    `[`     | the left bracket symbol                                                                                  |
|    2     |     0      | `$ident$`  | a [variable] name                                                                                        |
|    3     |     1      | `$symbol$` | the operator                                                                                             |
|    4     |     2      |  `$int$`   | an integer number                                                                                        |
|    5     |            |    `]`     | the right bracket symbol                                                                                 |
|    6     |            |    `<-`    | the left-arrow symbol (which is a [reserved symbol]({{rootUrl}}/appendix/operators.md#symbols) in Rhai). |
|    7     |     3      |  `$expr$`  | an expression, which may be enclosed with `{` ... `}`, or not.                                           |
|    8     |            |    `:`     | the colon symbol                                                                                         |
|    9     |     4      | `$block$`  | a statements block, which must be enclosed with `{` ... `}`.                                             |

This syntax matches the following sample code and generates five inputs (one for each non-keyword):

```rust
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

The signature of an implementation function is as follows.

> ```rust
> Fn(context: &mut EvalContext, inputs: &[Expression]) -> Result<Dynamic, Box<EvalAltResult>>
> ```

where:

| Parameter |                Type                 | Description                                           |
| --------- | :---------------------------------: | ----------------------------------------------------- |
| `context` | [`&mut EvalContext`][`EvalContext`] | mutable reference to the current _evaluation context_ |
| `inputs`  |           `&[Expression]`           | a list of input expression trees                      |

and [`EvalContext`] is a type that encapsulates the current _evaluation context_.

#### Return value

Return value is the result of evaluating the custom syntax expression.

#### Access arguments

The most important argument is `inputs` where the matched identifiers (`$ident$`), expressions/statements (`$expr$`)
and statements blocks (`$block$`) are provided.

To access a particular argument, use the following patterns:

| Argument type | Pattern (`n` = slot in `inputs`)                                                                             |             Result type             | Description           |
| :-----------: | ------------------------------------------------------------------------------------------------------------ | :---------------------------------: | --------------------- |
|   `$ident$`   | `inputs[n].get_string_value().unwrap()`                                                                      |               `&str`                | [variable] name       |
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

```rust
let expression = &inputs[0];

// Use 'get_literal_value' with a turbo-fish type to extract the value
let string_value = expression.get_literal_value::<ImmutableString>().unwrap();
let string_slice = expression.get_string_value().unwrap();

let float_value = expression.get_literal_value::<FLOAT>().unwrap();

// Or assign directly to a variable with type...
let int_value: i64 = expression.get_literal_value().unwrap();

// Or use type inference!
let bool_value = expression.get_literal_value().unwrap();

if bool_value { ... }       // 'bool_value' inferred to be 'bool'
```

#### Evaluate an expression tree

Use the `EvalContext::eval_expression_tree` method to evaluate an arbitrary expression tree
within the current evaluation context.

```rust
let expression = &inputs[0];
let result = context.eval_expression_tree(expression)?;
```

#### Retain variables in block scope

When an expression tree actually contains a statements block (i.e. `$block`), local
[variables]/[constants] defined within that block are usually removed at the end of the block.

Sometimes it is useful to retain these local [variables]/[constants] for further processing
(e.g. collecting new [variables] into an [object map]).

As such, evaluate the expression tree using the `EvalContext::eval_expression_tree_raw` method which
contains a parameter to control whether the statements block should be rewound.

```rust
// Assume 'expression' contains a statements block with local variable definitions
let expression = &inputs[0];
let result = context.eval_expression_tree_raw(expression, false)?;

// Variables defined within 'expression' persist in context.scope()
```

#### Declare variables

New [variables]/[constants] maybe declared (usually with a [variable] name that is passed in via `$ident$`).

It can simply be pushed into the [`Scope`].

```rust
let var_name = inputs[0].get_string_value().unwrap();
let expression = &inputs[1];

context.scope_mut().push(var_name, 0_i64);      // declare new variable

let result = context.eval_expression_tree(expression)?;
```


### Step Three &ndash; Register the Custom Syntax

Use `Engine::register_custom_syntax` to register a custom syntax.

Again, beware that the _first_ symbol must be unique.  If there already exists a custom syntax starting
with that symbol, the previous syntax will be overwritten.

The syntax is passed simply as a slice of `&str`.

```rust
// Custom syntax implementation
fn implementation_func(context: &mut EvalContext, inputs: &[Expression]) -> Result<Dynamic, Box<EvalAltResult>> {
    let var_name = inputs[0].get_string_value().unwrap();
    let stmt = &inputs[1];
    let condition = &inputs[2];

    // Push new variable into the scope BEFORE 'context.eval_expression_tree'
    context.scope_mut().push(var_name.to_string(), 0_i64);

    let mut count = 0_i64;

    loop {
        // Evaluate the statements block
        context.eval_expression_tree(stmt)?;

        count += 1;

        // Declare a new variable every three turns...
        if count % 3 == 0 {
            context.scope_mut().push(format!("{var_name}{count}"), count);
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
    [ "exec", "<", "$ident$", ">", "->", "$block$", "while", "$expr$" ], // the custom syntax
    true,  // variables declared within this custom syntax
    implementation_func
)?;
```

Remember that a custom syntax acts as an _expression_, so it can show up practically anywhere:

```rust
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
and [operators] that should not be used by the user.  The would leave only the bare minimum
language surface exposed, together with the custom syntax that is tailor-designed for
the scenario.

A [keyword] or [operator] that is disabled can still be used in a custom syntax.

In an extreme case, it is possible to disable _every_ [keyword] in the language, leaving only
custom syntax (plus possibly expressions).  But again, Don't Do It™ &ndash; unless you are certain
of what you're doing.


### Step Five &ndash; Document

For custom syntax, documentation is crucial.

Make sure there are _lots_ of examples for users to follow.


### Step Six &ndash; Profit!


Practical Example &ndash; Matrix Literal
----------------------------------------

Say you'd want to use something like [`ndarray`](https://crates.io/crates/ndarray) to manipulate matrices.

However, you'd like to write matrix literals in a more intuitive syntax than an [array]
of [arrays].

In other words, you'd like to turn:

```rust
// Array of arrays
let matrix = [ [  a, b,     0 ],
               [ -b, a,     0 ],
               [  0, 0, c * d ] ];
```

into:

```rust
// Directly parse to an ndarray::Array (look ma, no commas!)
let matrix = @|  a   b   0  |
              | -b   a   0  |
              |  0   0  c*d |;
```

This can easily be done via a custom syntax, which yields a syntax that is more pleasing.

```rust
// Disable the '|' symbol since it'll conflict with the bit-wise OR operator.
// Do this BEFORE registering the custom syntax.
engine.disable_symbol("|");

engine.register_custom_syntax(
    ["@", "|", "$expr$", "$expr$", "$expr$", "|", 
          "|", "$expr$", "$expr$", "$expr$", "|",
          "|", "$expr$", "$expr$", "$expr$", "|" 
    ],
    false,
    |context, inputs| {
        use ndarray::arr2;

        let mut values = [[0.0; 3]; 3];

        for y in 0..3 {
            for x in 0..3 {
                let offset = y * 3 + x;

                match context.eval_expression_tree(&inputs[offset])?.as_float() {
                    Ok(v) => values[y][x] = v,
                    Err(typ) => return Err(Box::new(EvalAltResult::ErrorMismatchDataType(
                                            "float".to_string(), typ.to_string(),
                                            inputs[offset].position()
                                )))
                }
            }
        }

        let matrix = arr2(&values);

        Ok(Dynamic::from(matrix))
    },
)?;
```

For matrices of flexible dimensions, check out [custom syntax parsers](custom-syntax-parsers.md).


Practical Example &ndash; Defining Temporary Variables
------------------------------------------------------

It is possible to define temporary [variables]/[constants] which are available only to code blocks
within the custom syntax.

```rust
engine.register_custom_syntax(
    [ "with", "offset", "(", "$expr$", ",", "$expr$", ")", "$block$", ],
    true,   // must be true in order to define new variables
    |context, inputs| {
        // Get the two offsets
        let x = context.eval_expression_tree(&inputs[0])?.as_int().map_err(|typ| Box::new(
            EvalAltResult::ErrorMismatchDataType("integer".to_string(), typ.to_string(), inputs[0].position())
        ))?;
        let y = context.eval_expression_tree(&inputs[1])?.as_int().map_err(|typ| Box::new(
            EvalAltResult::ErrorMismatchDataType("integer".to_string(), typ.to_string(), inputs[1].position())
        ))?;

        // Add them as temporary constants into the scope, available only to the code block
        let orig_len = context.scope().len();

        context.scope_mut().push_constant("x", x);
        context.scope_mut().push_constant("y", y);

        // Run the code block
        let result = context.eval_expression_tree(&inputs[2]);

        // Remove the temporary constants from the scope so they don't leak outside
        context.scope_mut().rewind(orig_len);

        // Return the result
        result
    },
)?;
```

Practical Example &ndash; Recreating C's Ternary Operator
---------------------------------------------------------

Rhai has [if-expressions](../language/if.md#if-expression), but sometimes a C-style _ternary_ operator
is more concise.

```rust
// A custom syntax must start with a unique symbol, so we use 'iff'.
// Register the custom syntax: iff condition ? true-value : false-value
engine.register_custom_syntax(
    ["iff", "$expr$", "?", "$expr$", ":", "$expr$"],
    false,
    |context, inputs| match context.eval_expression_tree(&inputs[0])?.as_bool() {
        Ok(true) => context.eval_expression_tree(&inputs[1]),
        Ok(false) => context.eval_expression_tree(&inputs[2]),
        Err(typ) => Err(Box::new(EvalAltResult::ErrorMismatchDataType(
            "bool".to_string(), typ.to_string(), inputs[0].position()
        ))),
    },
)?;
```

```admonish tip.small "Tip: Custom syntax performance"

The code in the example above is essentially what the [`if`] statement does internally, and since
custom syntax is pre-parsed, there really is no performance penalty!
```


Practical Example &ndash; Recreating JavaScript's `var` Statement
-----------------------------------------------------------------

The following example recreates a statement similar to the `var` variable declaration syntax in
JavaScript, which creates a global variable if one doesn't already exist.
There is currently no equivalent in Rhai.

```rust
// Register the custom syntax: var x = ???
engine.register_custom_syntax([ "var", "$ident$", "=", "$expr$" ], true, |context, inputs| {
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
