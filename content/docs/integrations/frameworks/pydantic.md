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

## Working with Nested LanceModels

While LanceDB provides robust support for a variety of data types, it's important to understand the limitations and best practices when working with nested LanceModels. This section will guide you through the process of using nested LanceModels and address some common issues you might encounter.

### Understanding LanceModel Field Types

LanceDB integrates with Pydantic for schema inference, data ingestion, and query result casting. This means you can define your data models using Pydantic's `BaseModel` and LanceDB's `LanceModel`. However, not all Pydantic field types are currently supported by LanceDB.

When defining your LanceModel, you can use the following field types:

- `int`: Converted to `pyarrow.int64`
- `float`: Converted to `pyarrow.float64`
- `bool`: Converted to `pyarrow.bool`
- `str`: Converted to `pyarrow.utf8()`
- `list`: Converted to `pyarrow.ListType`

### Nested LanceModels

You might be tempted to use a `list[LanceModel]` as a field type in your LanceModel. However, LanceDB does not currently support this type of nested LanceModel. If you try to do so, you will encounter a `TypeError` when attempting to convert the Pydantic type to an Arrow Type.

Here's an example of what not to do:

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

### Best Practices for Handling Nested LanceModels

If you need to create a LanceModel that includes a list of other LanceModels, you should instead use a `list[dict]` field type. Each dictionary in the list can then be used to instantiate the nested LanceModel.

Here's an example of how to properly handle nested LanceModels:

```python
from lancedb.pydantic import LanceModel

class SubFeature1(LanceModel):
    amount: int
    name: str

class RandomFeature1(LanceModel):
    email: str
    items: list[dict]  # Use this instead of list[LanceModel]

# You can then instantiate RandomFeature1 with a list of SubFeature1 instances
random_feature1 = RandomFeature1(email="test@example.com", items=[SubFeature1(amount=10, name="item1").dict(), SubFeature1(amount=20, name="item2").dict()])
```

In this example, `items` is a list of dictionaries, each of which can be used to instantiate a `SubFeature1` instance. This approach allows you to work with nested LanceModels without encountering any type conversion errors.

### Troubleshooting

If you encounter a `TypeError` when working with LanceModels, the first thing to check is your field types. Make sure you are only using supported field types and that you are not trying to use a `list[LanceModel]` as a field type.

If you're still having trouble, don't hesitate to reach out to the LanceDB community for help. We're always here to assist you in unlocking the full potential of LanceDB.