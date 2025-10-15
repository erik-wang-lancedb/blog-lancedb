---
title: "Working with Nested LanceModel Objects in LanceDB"
description: "Learn how to use nested LanceModel objects as field types in LanceDB and understand the limitations."
weight: 5
---

LanceDB is a powerful vector database for AI applications, and it integrates seamlessly with Pydantic for schema inference, data ingestion, and query result casting. However, when using LanceDB with Pydantic, you might encounter some limitations, especially when dealing with nested LanceModel objects.

In this guide, we will discuss how to use nested LanceModel objects as field types in LanceDB and provide some practical examples in Python and TypeScript.

## Understanding the Limitation

Currently, LanceDB does not support converting Pydantic custom types, including nested LanceModel objects, to Apache Arrow DataType. This limitation is due to LanceDB's internal data storage format, which is Apache Arrow.

When you try to use a `list[LanceModel]` as a field type in LanceDB, you might encounter a `TypeError` like this:

```python
TypeError: Converting Pydantic type to Arrow Type: unsupported type <class '__main__.SubFeature1'>.
```

This error occurs because LanceDB cannot convert the custom Pydantic type `SubFeature1` to an Apache Arrow DataType.

## Working Around the Limitation

While LanceDB does not directly support nested LanceModel objects, you can work around this limitation by flattening the nested LanceModel objects into separate fields in the parent LanceModel. Here's an example:

```python
from lancedb.pydantic import LanceModel

class SubFeature1(LanceModel):
    amount: int
    name: str

class RandomFeature1(LanceModel):
    email: str
    item_amount: int
    item_name: str
```

In this example, we flattened the `SubFeature1` fields into the `RandomFeature1` model. Now, `RandomFeature1` can be used in LanceDB without any issues.

## Troubleshooting

If you encounter a `TypeError` when using nested LanceModel objects in LanceDB, check if you are trying to use a `list[LanceModel]` as a field type. If so, try flattening the nested LanceModel objects into separate fields in the parent LanceModel.

## Conclusion

While LanceDB's integration with Pydantic provides many benefits, it also comes with some limitations, especially when dealing with nested LanceModel objects. By understanding these limitations and knowing how to work around them, you can effectively use LanceDB with Pydantic in your AI applications.