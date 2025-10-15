---
title: "Pydantic"
sidebar_title: "Pydantic"
weight: 3
---

[Pydantic](https://docs.pydantic.dev/latest/) is a data validation library in Python.
LanceDB integrates with Pydantic for schema inference, data ingestion, and query result casting.
Using `lancedb.pydantic.LanceModel`, users can seamlessly
integrate Pydantic with the rest of the LanceDB APIs.

First, import the necessary LanceDB and Pydantic modules:

{{< code language="python" source="examples/py/test_pydantic_integration.py" id="imports" />}}

Next, define your Pydantic model by inheriting from `LanceModel` and specifying your fields including a vector field:

{{< code language="python" source="examples/py/test_pydantic_integration.py" id="base_model" />}}

Set the database connection URL:

{{< code language="python" source="examples/py/test_pydantic_integration.py" id="set_url" />}}

Now you can create a table, add data, and perform vector search operations:

{{< code language="python" source="examples/py/test_pydantic_integration.py" id="base_example" />}}


## Vector Field

LanceDB provides a `lancedb.pydantic.Vector` method to define a
vector Field in a Pydantic Model.

```python
>>> import pydantic
>>> from lancedb.pydantic import Vector
...
>>> class MyModel(pydantic.BaseModel):
...     id: int
...     url: str
...     embeddings: Vector(768)
>>> schema = pydantic_to_schema(MyModel)
>>> assert schema == pa.schema([
...     pa.field("id", pa.int64(), False),
...     pa.field("url", pa.utf8(), False),
...     pa.field("embeddings", pa.list_(pa.float32(), 768))
... ])
```

This example demonstrates how LanceDB automatically converts Pydantic field types to their corresponding Apache Arrow data types. The `pydantic_to_schema()` function takes a Pydantic model and generates an Arrow schema where:
- `int` fields become `pa.int64()` (64-bit integers)
- `str` fields become `pa.utf8()` (UTF-8 encoded strings)  
- `Vector(768)` becomes `pa.list_(pa.float32(), 768)` (fixed-size list of 768 float32 values)
- The `False` parameter indicates that the fields are not nullable

## Type Conversion

LanceDB automatically convert Pydantic fields to
[Apache Arrow DataType](https://arrow.apache.org/docs/python/generated/pyarrow.DataType.html#pyarrow.DataType).

Current supported type conversions:

| Pydantic Field Type | PyArrow Data Type |
| ------------------- | ----------------- |
| `int`               | `pyarrow.int64`   |
| `float`              | `pyarrow.float64`  |
| `bool`              | `pyarrow.bool`    |
| `str`               | `pyarrow.utf8()`    |
| `list`              | `pyarrow.List`    |
| `BaseModel`         | `pyarrow.Struct`    |
| `Vector(n)`         | `pyarrow.FixedSizeList(float32, n)` |

LanceDB supports to create Apache Arrow Schema from a
`pydantic.BaseModel`
via `lancedb.pydantic.pydantic_to_schema` method.

```python
>>> from typing import List, Optional
>>> import pydantic
>>> from lancedb.pydantic import pydantic_to_schema, Vector
>>> class FooModel(pydantic.BaseModel):
...     id: int
...     s: str
...     vec: Vector(1536)  # fixed_size_list<item: float32>[1536]
...     li: List[int]
...
>>> schema = pydantic_to_schema(FooModel)
>>> assert schema == pa.schema([
...     pa.field("id", pa.int64(), False),
...     pa.field("s", pa.utf8(), False),
...     pa.field("vec", pa.list_(pa.float32(), 1536)),
...     pa.field("li", pa.list_(pa.int64()), False),
... ])
```

This example shows a more complex Pydantic model with various field types and demonstrates how LanceDB handles:
- Basic types: `int` and `str` fields
- Vector fields: `Vector(1536)` creates a fixed-size list of 1536 float32 values
- List fields: `List[int]` becomes a variable-length list of int64 values
- Schema generation: The `pydantic_to_schema()` function automatically converts all these types to their Arrow equivalents

---

## bug(python): Can not use list[LanceModel] inside LanceModel

---
title: "Using Nested LanceModels in LanceDB"
description: "A guide to using nested LanceModels in LanceDB, handling unsupported types, and troubleshooting common issues."
weight: 20
---

# Introduction

LanceDB is a powerful vector database for AI applications, built on top of the Lance columnar data format. It integrates with Pydantic for schema inference, data ingestion, and query result casting. However, using nested LanceModels or `list[LanceModel]` as a field type in LanceDB might cause some issues due to unsupported types. This guide will help you understand how to use nested LanceModels, handle unsupported types, and troubleshoot common issues.

## Using Nested LanceModels

You can use nested Pydantic models to represent complex data structures in LanceDB. Here is an example:

```python
from lancedb.pydantic import LanceModel

class SubFeature(LanceModel):
    amount: int
    name: str

class MainFeature(LanceModel):
    email: str
    items: list[SubFeature]
```

In this example, `MainFeature` is a LanceModel that contains a list of `SubFeature` LanceModels. However, you might encounter a `TypeError` when trying to convert this Pydantic type to an Arrow Type.

## Handling Unsupported Types

LanceDB automatically converts Pydantic fields to Apache Arrow DataType. The current supported type conversions include `int`, `float`, `bool`, and `str`. When using a `list` as a field type, LanceDB converts it to `pyarrow.ListType()`. However, LanceDB does not yet support converting Pydantic custom types, such as `list[LanceModel]`.

If you encounter a `TypeError` like the following:

```python
TypeError: Converting Pydantic type to Arrow Type: unsupported type <class '__main__.SubFeature'>.
```

This means that LanceDB is unable to convert the `SubFeature` LanceModel to an Arrow Type. If you need to use a `list[LanceModel]` as a field type, you can file a feature request on the LanceDB Github repo.

## Troubleshooting Common Issues

If you encounter issues when using nested LanceModels in LanceDB, here are some troubleshooting steps:

1. **Check the field types:** Make sure all the field types in your LanceModels are supported by LanceDB. Unsupported types can cause a `TypeError`.

2. **Update LanceDB:** Make sure you are using the latest version of LanceDB. Some issues might have been fixed in newer versions.

3. **File a feature request:** If you need to use a feature that is not currently supported by LanceDB, such as using `list[LanceModel]` as a field type, you can file a feature request on the LanceDB Github repo.

Remember, LanceDB is a powerful tool for handling complex data structures. By understanding how to use nested LanceModels and handle unsupported types, you can make the most of LanceDB's capabilities.