The Rhai Book
=============

{{#title The Rhai Book}}

{{#include ../links.md}}

![Rhai Logo]({{rootUrl}}/images/logo/rhai-banner-transparent-colour.svg)

Rhai is an embedded scripting language and evaluation engine for Rust that gives a safe and easy way
to add scripting to any application.


Versions
--------

This Book is for version **{{version}}** of Rhai.

{% if rootUrl != "" and not rootUrl is ending_with("vnext") %}
For the latest development version, see [here]({{rootUrl}}/vnext/).
{% endif %}


Trivia
------

```admonish question "Etymology of the name \\"Rhai\\""

In the beginning there was [ChaiScript](http://chaiscript.com),
which is an embedded scripting language for C++.
Originally it was intended to be a scripting language similar to **JavaScript**.

With java being a kind of hot beverage, the new language was named after
another hot beverage &ndash; [**Chai**](https://en.wikipedia.org/wiki/Chai),
which is the word for "tea" in many world languages and, in particular,
a popular kind of [spicy milk tea consumed in India](https://en.wikipedia.org/wiki/Masala_chai).

Later, when the novel implementation technique behind ChaiScript was ported from C++ to Rust,
logically the `C` was changed to an `R` to make it "RhaiScript", or just "Rhai".

&ndash; Rhai author [Johnathan Turner](https://github.com/jntrnr)
```

```admonish question "Origin of the Rhai logo"

<div style="float:right;text-align:center;width:20%">
    <img src="{{rootUrl}}/images/logo/rhai_logo_old.png" title="Prototype Rhai logo">
    <div style="font-size:60%">Original prototype</div>
</div>

One of Rhai's maintainers, [`@schungx`](https://github.com/schungx), was thinking about a logo
when he accidentally came across a copy of _Catcher in the Rye_ in a restaurant, and drew the
first prototype version of the logo.

Then [`@semirix`](https://github.com/semirix) refined it to the current version.
```

~~~admonish question "The \`rhai.rs\` domain"

[`@yrashk`](https://github.com/yrashk) sponsored the domain [`rhai.rs`](https://rhai.rs).
~~~
