---
title: "Pydantic"
sidebar_title: "Pydantic"
weight: 3
---

[Pydantic](https://docs.pydantic.dev/latest/) is a data validation library in Python. LanceDB integrates with Pydantic for schema inference, data ingestion, and query result casting. Using `lancedb.pydantic.LanceModel`, users can seamlessly integrate Pydantic with the rest of the LanceDB APIs.

First, import the necessary LanceDB and Pydantic modules:

{{< code language="python" source="examples/py/test_pydantic_integration.py" id="imports" />}}

Next, define your Pydantic model by inheriting from `LanceModel` and specifying your fields including a vector field:

{{< code language="python" source="examples/py/test_pydantic_integration.py" id="base_model" />}}

Set the database connection URL:

{{< code language="python" source="examples/py/test_pydantic_integration.py" id="set_url" />}}

Now you can create a table, add data, and perform vector search operations:

{{< code language="python" source="examples/py/test_pydantic_integration.py" id="base_example" />}}

## Vector Field

LanceDB provides a `lancedb.pydantic.Vector` method to define a vector Field in a Pydantic Model.

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

## Working with Nested LanceModels in LanceDB

title: "Working with Nested LanceModels in LanceDB"
description: "Learn how to properly use nested LanceModels in LanceDB and troubleshoot common issues."
weight: 5

## Introduction

LanceDB is a powerful tool for managing and querying large datasets. It integrates with Pydantic, a data validation library in Python, for schema inference, data ingestion, and query result casting. However, there are certain limitations and considerations when using nested Pydantic models in LanceDB, particularly when they do not contain any vector fields. This guide will help you understand how to use nested LanceModels effectively and troubleshoot common issues.

## Using Nested LanceModels

Nested LanceModels are a way to represent complex data structures in LanceDB. Here's an example of how you can define nested LanceModels:

{{< code language="python" >}}
from lancedb.pydantic import LanceModel

class SubFeature(LanceModel):
    amount: int
    name: str

class MainFeature(LanceModel):
    email: str
    items: list[SubFeature]
{{< /code >}}

In this example, `MainFeature` includes a list of `SubFeature` objects. This is a powerful way to model complex data structures, but it comes with certain limitations.

## Limitations and Considerations

Currently, LanceDB does not support converting Pydantic custom types, including nested LanceModels, to Apache Arrow DataTypes. When you try to convert a LanceModel that includes a list of other LanceModels to an Arrow schema, you might encounter a `TypeError`:

{{< code language="python" >}}
TypeError: Converting Pydantic type to Arrow Type: unsupported type <class '__main__.SubFeature'>.
{{< /code >}}

This error occurs because LanceDB cannot automatically convert the `SubFeature` class to an Arrow DataType. Currently, the supported type conversions are `int`, `float`, `bool`, and `str`.

## Workaround

While we are working on adding support for nested LanceModels, there is a workaround you can use. Instead of using a list of LanceModels, consider flattening your data structure or using supported data types that can be easily converted to Arrow DataTypes. This may involve restructuring your data model to fit within the current limitations of LanceDB's Arrow conversion capabilities.