Multiple Instantiation
======================

{{#include ../links.md}}

Rhai's [features] are not strictly additive.  This is easily deduced from the [`no_std`] feature
which prepares the crate for `no-std` builds.  Obviously, turning on this feature has a material
impact on how Rhai behaves.

Many crates resolve this by going the opposite direction: build for `no-std` in default, but add a
`std` feature, included by default, which builds for the `stdlib`.

Rhai, however, is more complex.


Rhai Language Features Are Not Additive
--------------------------------------

Language features cannot be easily made _additive_.

That is because the _lack_ of a language feature is a feature by itself.

```admonish question "A practical illustration"

Assume an _additive_ feature called `floating-point` that adds floating-point support.

Assume also that the application _omits_ the `floating-point` feature (why? perhaps integers are all
that make sense within the project domain). Floating-point numbers do not even parse under this
configuration and will generate syntax errors.

Now, assume that a dependent crate _also_ depends on Rhai, but a new version suddenly decides to
_require_ floating-point support. That dependent crate would, naturally, specify the
`floating-point` feature.

Under such circumstances, unless _exact_ versioning is used and the dependent crate depends on a
_different_ version of Rhai, Cargo automatically _merges_ both dependencies, with the `floating-point`
feature turned on because features are _additive_.

This will in turn break the application, which by itself specifically omits `floating-point` and
expects floating-point numbers to be rejected, in unexpected ways. Suddenly and without warning,
floating-point numbers show up in data which the application is not prepared to handle.

There is no way out of this dilemma, because the _lack_ of a language feature can be depended upon
as a feature.
```


Multiple Instantiations of Rhai Within The Same Project
------------------------------------------------------

The trick is to differentiate between multiple identical copies of Rhai, each having
a different [features] set, by their _sources_:

* Different versions from [`crates.io`](https://crates.io/crates/rhai/) &ndash; The official crate.

* Different releases from [`GitHub`](https://github.com/rhaiscript/rhai) &ndash; Crate source on GitHub.

* Forked copy of [https://github.com/rhaiscript/rhai](https://github.com/rhaiscript/rhai) on GitHub.

* Local copy of [https://github.com/rhaiscript/rhai](https://github.com/rhaiscript/rhai) downloaded form GitHub.

Use the following configuration in `Cargo.toml` to pull in multiple copies of Rhai within the same project:

```toml
[dependencies]
rhai = { version = "{{version}}", features = [ "no_float" ] }
rhai_github = { git = "https://github.com/rhaiscript/rhai", features = [ "unchecked" ] }
rhai_my_github = { git = "https://github.com/my_github/rhai", branch = "variation1", features = [ "serde", "no_closure" ] }
rhai_local = { path = "../rhai_copy" }
```

The example above creates four different modules: `rhai`, `rhai_github`, `rhai_my_github` and
`rhai_local`, each referring to a different Rhai copy with the appropriate [features] set.

Only one crate of any particular version can be used from each source, because Cargo merges all
candidate cases within the same source, adding all [features] together.

If more than four different instantiations of Rhai is necessary (why?), create more local
repositories or GitHub forks or branches.


```admonish danger.small "No way To avoid dependency conflicts"

Unfortunately, pulling in Rhai from different sources do not resolve the problem of [features]
conflict between dependencies.  Even overriding `crates.io` via the `[patch]` manifest section
doesn't work &ndash; all dependencies will eventually find the only one copy.

What is necessary &ndash; multiple copies of Rhai, one for each dependent crate that requires it,
together with their _unique_ [features] set intact.  In other words, turning off Cargo's crate
merging feature _just for Rhai_.

Unfortunately, as of this writing, there is no known method to achieve it.

Therefore, moral of the story: avoid pulling in multiple crates that depend on Rhai.
```
