Serialize an AST
================

{{#include ../links.md}}

In many situations, it is tempting to _serialize_ an [`AST`], so that it can be loaded and recreated
later on.

In Rhai, there is usually little reason to do so.

```admonish failure.small "Don't Do This"

Serialize the [`AST`] into some format for storage.
```

```admonish success.small "Do This"

Store a copy of the original script, preferably compressed.
```

Storing the original script text, preferably compressed (via `gzip` etc.) usually yields much smaller data size.

Plus, is it possibly faster to recompile the original script than to recreate the [`AST`] via
deserialization.

That is because the deserialization processing is essentially a form of parsing, in this case
parsing the serialized data into an [`AST`] &ndash; an equivalent process to what Rhai does, which
is parsing the script text into the same [`AST`].


Illustration
------------

The following script occupies only 42 bytes, possibly less if compressed.
That is only slightly more than 5 words on a 64-bit CPU!

```rust
fn sum(x, y) { x + y }
print(sum(42, 1));
```

The [`AST`] would be _much_ more complicated and looks something like this:

```json
FnDef {
    Name: "sum",
    ThisType: None,
    Parameters: [
        "x",
        "y"
    ],
    Body: Block [
        Return {
            Expression {
                FnCall {
                    Name: "+",
                    Arguments: [
                        Variable "x",
                        Variable "y"
                    ]
                }
            }
        }
    ]
}
Block [
    FnCall {
        Name: "print",
        Arguments: [
            Expression {
                FnCall {
                    Name: "sum",
                    Arguments: [
                        Constant 42,
                        Constant 1
                    ]
                }
            }
        ]
    }
]
```

which would take _much_ more space to serialize.

For instance, the constant 1 alone would take up 8 bytes, while the script text takes up only one byte!
