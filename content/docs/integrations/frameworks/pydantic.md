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

## Understanding LanceModel Field Types

In LanceDB, the `LanceModel` is a powerful tool for defining your data schema. It integrates with Pydantic for schema inference, data ingestion, and query result casting. However, it's important to understand the limitations and specific use cases of `LanceModel` field types to avoid errors and ensure smooth operation.

### Supported Field Types

LanceDB automatically converts Pydantic fields to Apache Arrow DataType. The currently supported type conversions include:

- `int` to `pyarrow.int64`
- `float` to `pyarrow.float64`
- `bool` to `pyarrow.bool`
- `str` to `pyarrow.utf8()`
- `list` to `pyarrow.ListType`

These conversions allow you to define your data schema in a Pythonic way while leveraging the performance benefits of Apache Arrow's columnar data format.

### Unsupported Field Types

Currently, LanceDB does not support converting Pydantic custom types, including a `list` of `LanceModel`. This means you cannot use a `list[LanceModel]` as a field type inside another `LanceModel`. If you try to do so, you will encounter a `TypeError` indicating an unsupported type conversion.

For example, the following code will raise an error:

```python
from lancedb.pydantic import LanceModel

class SubFeature1(LanceModel):
    amount: int
    name: str

class RandomFeature1(LanceModel):
    email: str
    items: list[SubFeature1]  # This is not supported

print(RandomFeature1.to_arrow_schema())
```

### Workarounds

While `list[LanceModel]` is not currently supported, there are alternative ways to structure your data to achieve similar results. One possible workaround is to flatten your data structure and use a `Vector` field to store complex data types.

```python
from lancedb.pydantic import LanceModel, Vector

class RandomFeature1(LanceModel):
    email: str
    items: Vector  # This is supported
```

In this example, the `Vector` field can store a list of complex data types, including custom models. However, keep in mind that this workaround may not be suitable for all use cases, and you should carefully consider the structure of your data before deciding on a solution.

### Requesting New Features

If you find that LanceDB does not support a feature you need, such as converting a `list[LanceModel]` to an Arrow type, you can file a feature request on the LanceDB Github repo. The LanceDB team is always open to feedback and suggestions to improve the product.

## Troubleshooting

If you encounter a `TypeError` when using a `LanceModel`, check your field types to ensure they are supported by LanceDB. If you're using a custom type or a `list[LanceModel]`, consider restructuring your data or using a `Vector` field instead.

Remember, LanceDB is built on top of the Lance columnar data format, which provides the foundation for its multimodal capabilities. Understanding this underlying architecture can help you troubleshoot issues and make the most of LanceDB's features.