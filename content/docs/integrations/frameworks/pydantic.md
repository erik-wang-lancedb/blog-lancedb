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

## Understanding LanceModel and List Field Type Limitations

LanceDB integrates with Pydantic for schema inference, data ingestion, and query result casting. However, it's important to note that LanceDB does not yet fully support all Pydantic custom types. One of these limitations is the use of `list[LanceModel]` as a field type within a LanceModel.

When you attempt to use `list[LanceModel]` as a field type, you may encounter a TypeError, as LanceDB cannot convert this Pydantic custom type to Apache Arrow DataType. This error occurs because LanceDB internally stores data in Apache Arrow format and currently supports a limited set of type conversions.

```python
from lancedb.pydantic import LanceModel

class SubFeature1(LanceModel):
    amount: int
    name: str

class RandomFeature1(LanceModel):
    email: str
    items: list[SubFeature1]  # Unsupported type

print(RandomFeature1.to_arrow_schema())
```

This code will result in a TypeError:

```
TypeError: Converting Pydantic type to Arrow Type: unsupported type <class '__main__.SubFeature1'>.
```

## Working Around the Limitation

While LanceDB does not currently support `list[LanceModel]` as a field type, there are alternative approaches you can consider.

One approach is to flatten your data structure. Instead of nesting LanceModels within a list, consider creating a separate LanceModel for each item in the list. This approach can help you avoid the TypeError and ensure that your data is stored correctly in LanceDB.

Another approach is to use supported field types within your LanceModel. LanceDB currently supports the following type conversions:

- int to pyarrow.int64
- float to pyarrow.float64
- bool to pyarrow.bool
- str to pyarrow.utf8()
- list to pyarrow.ListType

By using these supported field types, you can ensure that your LanceModels are compatible with LanceDB.

## Conclusion

While LanceDB's integration with Pydantic provides powerful functionality for schema inference, data ingestion, and query result casting, it's important to understand the limitations of this integration. By being aware of these limitations and knowing how to work around them, you can make the most of LanceDB's features and avoid potential issues.