Disable Custom Types
====================

{{#include ../links.md}}


`no_object` Feature
-------------------

The custom types API `register_type`, `register_type_with_name`, `register_get`, `register_get_result`,
`register_set`, `register_set_result` and `register_get_set` are not available under [`no_object`].


`no_index` Feature
------------------

The indexers API `register_indexer_get`, `register_indexer_get_result`, `register_indexer_set`,
`register_indexer_set_result`, and `register_indexer_get_set` are not available under
[`no_object`]`+`[`no_index`].
