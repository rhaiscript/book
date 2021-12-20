Use Rhai as a Domain-Specific Language (DSL)
===========================================

{{#include ../links.md}}

Rhai can be successfully used as a domain-specific language (DSL).


Expressions Only
----------------

In many DSL scenarios, only evaluation of expressions is needed.

The [`Engine::eval_expression_XXX`][`eval_expression`] API can be used to restrict
a script to expressions only.


Unicode Standard Annex #31 Identifiers
-------------------------------------

Variable names and other identifiers do not necessarily need to be ASCII-only.

The [`unicode-xid-ident`] feature, when turned on, causes Rhai to allow variable names and identifiers
that follow [Unicode Standard Annex #31](http://www.unicode.org/reports/tr31/).

This is sometimes useful in a non-English DSL.


Disable Keywords and/or Operators
--------------------------------

In some DSL scenarios, it is necessary to further restrict the language to exclude certain
language features that are not necessary or dangerous to the application.

For example, a DSL may disable the [`while`] loop while keeping all other statement types intact.

It is possible, in Rhai, to surgically [disable keywords and operators].


Custom Operators
----------------

On the other hand, some DSL scenarios require special operators that make sense only for
that specific environment.  In such cases, it is possible to define [custom operators] in Rhai.

```rust no_run
let animal = "rabbit";
let food = "carrot";

animal eats food            // custom operator 'eats'

eats(animal, food)          // <- the above actually de-sugars to this

let x = foo # bar;          // custom operator '#'

let x = #(foo, bar)         // <- the above actually de-sugars to this
```

Although a [custom operator] always de-sugars to a simple function call,
nevertheless it makes the DSL syntax much simpler and expressive.


Custom Syntax
-------------

For advanced DSL scenarios, it is possible to define entire expression [_syntax_][custom syntax] &ndash;
essentially custom statement types.

For example, the following is a SQL-like syntax for some obscure DSL operation:

```rust no_run
let table = [..., ..., ..., ...];

// Syntax = calculate $ident$ ( $expr$ -> $ident$ ) => $ident$ : $expr$
let total = calculate sum(table->price) => row : row.weight > 50;

// Note: There is nothing special about those symbols; to make it look exactly like SQL:
// Syntax = SELECT $ident$ ( $ident$ ) AS $ident$ FROM $expr$ WHERE $expr$
let total = SELECT sum(price) AS row FROM table WHERE row.weight > 50;
```

After registering this custom syntax with Rhai, it can be used anywhere inside a script as
a normal expression.

For its evaluation, the callback function will receive the following list of inputs:

* `inputs[0] = "sum"` - math operator
* `inputs[1] = "price"` - field name
* `inputs[2] = "row"` - loop variable name
* `inputs[3] = Expression(table)` - data source
* `inputs[4] = Expression(row.weight > 50)` - filter predicate

Other identifiers, such as `"calculate"`, `"FROM"`, as well as symbols such as `->` and `:` etc.,
are parsed in the order defined within the custom syntax.
