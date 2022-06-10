Get Scripted Functions  Metadata from AST
=========================================

{{#include ../links.md}}

Use [`AST::iter_functions`](https://docs.rs/rhai/latest/rhai/struct.AST.html#method.iter_functions)
to iterate through all the script-defined [functions] in an [`AST`].


`ScriptFnMetadata`
------------------

The type returned from the iterator is `ScriptFnMetadata` with the following fields:

| Field      |   Requires   |    Type     | Description                                                           |
| ---------- | :----------: | :---------: | --------------------------------------------------------------------- |
| `name`     |              |   `&str`    | Name of [function]                                                    |
| `params`   |              | `Vec<&str>` | Number of parameters                                                  |
| `access`   |              | `FnAccess`  | • `FnAccess::Public` (public)<br/>• `FnAccess::Private` ([`private`]) |
| `comments` | [`metadata`] | `Vec<&str>` | [Doc-comments], if any, one per line                                  |
