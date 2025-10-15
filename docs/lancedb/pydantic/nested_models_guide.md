---
title: "Working with Nested Models in LanceDB"
description: "A comprehensive guide on how to use nested models in LanceDB, including handling unsupported types and troubleshooting common issues."
weight: 5
---

## Introduction

LanceDB, a vector database for AI applications, integrates with Pydantic for schema inference, data ingestion, and query result casting. This document provides a comprehensive guide on how to use nested models in LanceDB, including handling unsupported types and troubleshooting common issues.

## Using Nested Models

Nested models or `list[LanceModel]` as a field type in LanceModel allows you to represent complex data structures. However, there are some limitations to using nested models in LanceDB. Currently, LanceDB does not support converting Pydantic custom types. If you need this feature, consider filing a feature request on the LanceDB Github repo.

Here's an example of how you can use nested models:

```python
from lancedb.pydantic import LanceModel

class SubFeature1(LanceModel):
    amount: int
    name: str

class RandomFeature1(LanceModel):
    email: str
    items: list[SubFeature1]
```

## Type Conversion

LanceDB automatically converts Pydantic fields to Apache Arrow DataType. The current supported type conversions are:

```python
int -> pyarrow.int64
float -> pyarrow.float64
bool -> pyarrow.bool
str -> pyarrow.utf8()
list -> pyarrow.ListType
```

## Troubleshooting

### TypeError: Unsupported Type

If you encounter a `TypeError: Converting Pydantic type to Arrow Type: unsupported type`, it means that LanceDB is trying to convert a Pydantic type that it does not support. This usually happens when you're using a custom type or a nested model that contains an unsupported type.

Here's an example of how this error might occur:

```python
print(RandomFeature1.to_arrow_schema())
```

This will result in:

```python
TypeError: Converting Pydantic type to Arrow Type: unsupported type <class '__main__.SubFeature1'>.
```

To resolve this issue, ensure that your nested models only contain supported types. If you need to use a type that is not currently supported, consider filing a feature request on the LanceDB Github repo.

## Conclusion

While nested models provide a way to represent complex data structures in LanceDB, there are some limitations to be aware of. By understanding these limitations and knowing how to troubleshoot common issues, you can effectively use nested models in your LanceDB applications.