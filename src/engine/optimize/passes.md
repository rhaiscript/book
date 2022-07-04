Optimization Passes
===================

{{#include ../../links.md}}

[Script optimization] is performed via multiple _passes_.
Each pass does a specific optimization.

The optimization is completed when no passes can simplify the [`AST`] any further.


Built-in Optimization Passes
----------------------------

| Pass                                       | Description                                       |
| ------------------------------------------ | ------------------------------------------------- |
| [Dead code elimination](dead-code.md)      | Eliminates code that cannot be reached            |
| [Constants propagation](constants.md)      | Replaces [constants] with values                  |
| [Compound assignments rewrite](rewrite.md) | Rewrites assignments into compound assignments    |
| [Eager operator evaluation](op-eval.md)    | Eagerly calls operators with [constant] arguments |
| [Eager function evaluation](eager.md)      | Eagerly calls functions with [constant] arguments |
