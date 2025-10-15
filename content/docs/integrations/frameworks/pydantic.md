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

## Working with Nested LanceModel Lists in LanceDB

LanceDB provides robust support for various data types, including nested lists. However, when working with nested lists of LanceModel objects, there are certain limitations and considerations you need to be aware of.

### Understanding the Issue

When defining a LanceModel, you might want to include a list of other LanceModel objects as a field. For example:

```python
from lancedb.pydantic import LanceModel

class SubFeature(LanceModel):
    amount: int
    name: str

class MainFeature(LanceModel):
    email: str
    items: list[SubFeature]
```

In this example, `items` is a list of `SubFeature` LanceModel objects. However, when you try to convert this to an Arrow schema using `MainFeature.to_arrow_schema()`, you may encounter a `TypeError` indicating that the conversion of the Pydantic type to the Arrow type is unsupported.

This is because LanceDB's current version does not support converting Pydantic custom types, such as a list of LanceModel objects, to Arrow types.

### Limitations of Pydantic Custom Types in LanceDB

LanceDB integrates with Pydantic for schema inference, data ingestion, and query result casting. However, the conversion of Pydantic custom types to Arrow types is not yet supported. This includes the conversion of a list of LanceModel objects.

The current supported type conversions include:

- `int` to `pyarrow.int64`
- `float` to `pyarrow.float64`
- `bool` to `pyarrow.bool`
- `str` to `pyarrow.utf8()`
- `list` to `pyarrow.ListType`

### Handling TypeError When Converting Pydantic Models to Arrow Types

If you encounter a `TypeError` when converting Pydantic models to Arrow types, it's likely due to the use of an unsupported type. To resolve this issue, consider the following steps:

1. Check the data types in your LanceModel. Ensure they are supported by LanceDB for conversion to Arrow types.
2. If you're using a list of LanceModel objects, consider restructuring your data. Instead of using a list of LanceModel objects, you might use a list of dictionaries, which is a supported data type.
3. If you need to use a list of LanceModel objects, consider filing a feature request on the LanceDB Github repo. The LanceDB team is always open to community suggestions for improving the product.

### Example: Using a List of Dictionaries Instead of a List of LanceModel Objects

Here's an example of how you might restructure your data to use a list of dictionaries instead of a list of LanceModel objects:

```python
from lancedb.pydantic import LanceModel

class MainFeature(LanceModel):
    email: str
    items: list[dict]
```

In this example, `items` is a list of dictionaries. Each dictionary can represent a `SubFeature` object, with keys and values corresponding to the fields of the `SubFeature` LanceModel.

Remember, when working with LanceDB, it's important to understand the limitations and capabilities of the system. While LanceDB offers powerful features and flexibility, certain data types and structures may not be supported. Always refer to the LanceDB documentation for the most accurate and up-to-date information.