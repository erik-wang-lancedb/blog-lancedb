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

---

title: "Handling Nested LanceModel Lists in LanceDB"
description: "A guide on how to work with nested LanceModel lists in LanceDB, including potential issues and workarounds."
weight: 20
---

## Introduction

LanceDB is a powerful vector database for AI applications. It integrates with Pydantic for schema inference, data ingestion, and query result casting. However, there are some nuances when it comes to handling nested lists of `LanceModel` objects, especially when these do not contain any `Vector` fields. This guide will help you understand how to work with such data structures and troubleshoot any issues you might encounter.

## Understanding the Issue

When defining a `LanceModel` with a field of type `list[LanceModel]`, you might encounter a `TypeError` like the following:

```python
TypeError: Converting Pydantic type to Arrow Type: unsupported type <class '__main__.SubFeature1'>.
```

This error occurs because LanceDB currently does not support converting Pydantic custom types, including nested lists of `LanceModel` objects that do not contain any `Vector` fields.

## Workaround

While full support for nested lists of `LanceModel` objects is not yet available, there is a workaround you can use. Instead of using a list, you can use a `Vector` field in your `LanceModel`. Here's an example:

```python
from lancedb.pydantic import LanceModel, Vector

class SubFeature1(LanceModel):
    amount: int
    name: str

class RandomFeature1(LanceModel):
    email: str
    items: Vector[SubFeature1]

print(RandomFeature1.to_arrow_schema())
```

This code will not raise a `TypeError`, as `Vector` fields are supported by LanceDB.

## Requesting Additional Features

If you need support for nested lists of `LanceModel` objects without `Vector` fields, consider filing a feature request on the LanceDB Github repo. The LanceDB team is always interested in hearing about use cases that are not currently supported.

## Conclusion

While LanceDB provides powerful capabilities for AI applications, understanding its current limitations and available workarounds is crucial for effective use. By leveraging `Vector` fields, you can bypass current limitations with nested `LanceModel` lists.