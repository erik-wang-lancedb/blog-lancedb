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

## Issue with Nested LanceModels

When you try to use a list of LanceModels as a field type inside another LanceModel, you may encounter a TypeError. This happens because LanceDB does not yet support converting Pydantic custom types, which includes LanceModels.

```python
from lancedb.pydantic import LanceModel

class SubFeature1(LanceModel):
    amount: int
    name: str

class RandomFeature1(LanceModel):
    email: str
    items: list[SubFeature1]  # This will cause a TypeError

print(RandomFeature1.to_arrow_schema())
```

Running the above code will result in the following error:

```python
TypeError: Converting Pydantic type to Arrow Type: unsupported type <class '__main__.SubFeature1'>.
```

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

If you encounter a `TypeError` when using nested LanceModel objects in LanceDB, check if you are trying to use a nested LanceModel as a field type. Consider flattening your data structure or creating separate LanceModels for each level of your data hierarchy and linking them using a unique identifier.

## Understanding LanceModel

LanceModel, built on top of Pydantic, is a data validation library in Python. LanceDB integrates with Pydantic for schema inference, data ingestion, and query result casting. LanceDB automatically converts Pydantic fields to Apache Arrow DataType. The currently supported type conversions include:

```python
int -> pyarrow.int64
float -> pyarrow.float64
bool -> pyarrow.bool
str -> pyarrow.utf8()
list -> pyarrow.ListType
```

By understanding these limitations and applying the suggested workarounds, you can effectively work with nested LanceModels within LanceDB.