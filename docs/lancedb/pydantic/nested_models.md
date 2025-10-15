---
title: "Working with Nested LanceModels in LanceDB"
description: "Learn how to define and use nested LanceModels in LanceDB, and understand the limitations and workarounds."
weight: 15
---

LanceDB is a powerful vector database that integrates seamlessly with Pydantic for schema inference, data ingestion, and query result casting. This guide will help you understand how to define and use nested LanceModels in LanceDB, and the current limitations and workarounds.

## Defining Nested LanceModels

Firstly, let's define a simple nested LanceModel. We'll create a `SubFeature1` model and a `RandomFeature1` model that contains a list of `SubFeature1` instances.

```python
from lancedb.pydantic import LanceModel

class SubFeature1(LanceModel):
    amount: int
    name: str

class RandomFeature1(LanceModel):
    email: str
    items: list[SubFeature1]
```

## Converting Pydantic Models to Arrow Types

LanceDB automatically converts Pydantic fields to Apache Arrow DataTypes. The current supported type conversions are:

- `int` to `pyarrow.int64`
- `float` to `pyarrow.float64`
- `bool` to `pyarrow.bool`
- `str` to `pyarrow.utf8()`
- `list` to `pyarrow.ListType`

However, LanceDB does not yet support converting nested LanceModels to Arrow types. If you try to convert a nested LanceModel to an Arrow type, you will encounter a `TypeError`:

```python
print(RandomFeature1.to_arrow_schema())
```

This will result in:

```python
TypeError: Converting Pydantic type to Arrow Type: unsupported type <class '__main__.SubFeature1'>.
```

## Limitations and Workarounds

Currently, LanceDB does not support lists of LanceModels that do not contain any Vector fields. If you need to use nested LanceModels, a workaround is to include at least one Vector field in your nested LanceModel.

```python
from lancedb.pydantic import LanceModel, Vector

class SubFeature1(LanceModel):
    amount: int
    name: str
    vector: Vector

class RandomFeature1(LanceModel):
    email: str
    items: list[SubFeature1]
```

Now, `RandomFeature1.to_arrow_schema()` will not raise a `TypeError`.

## Conclusion

While LanceDB provides powerful features for handling complex data structures, there are some limitations when it comes to nested LanceModels. This guide has shown you how to define and use nested LanceModels, and provided a workaround for the current limitation. As LanceDB continues to evolve, we hope to see more support for complex data structures in the future.

If you encounter any issues or have any feature requests, please feel free to reach out to the LanceDB community.