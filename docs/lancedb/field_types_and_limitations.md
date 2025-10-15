---
title: "Working with Nested LanceModel Instances in LanceDB"
description: "Understand how to use nested LanceModel instances in LanceDB, handle unsupported types, and troubleshoot common errors."
weight: 20
---

LanceDB is a powerful tool for managing and querying data, and its integration with Pydantic makes it even more versatile. However, there are some nuances to using nested LanceModel instances, particularly when they do not contain any Vector fields. This guide will help you understand these nuances and provide solutions to common issues.

## Understanding LanceModel and Pydantic Integration

LanceDB integrates with Pydantic for schema inference, data ingestion, and query result casting. Pydantic is a data validation library in Python that allows you to define data models with type annotations.

Here's a simple example of a Pydantic model:

```python
from pydantic import BaseModel

class User(BaseModel):
    name: str
    age: int
```

In LanceDB, you can use the `LanceModel` class, which is a subclass of Pydantic's `BaseModel`, to define your data models. `LanceModel` provides additional methods for working with LanceDB, such as `to_arrow_schema()`.

## Using Nested LanceModel Instances

You can define nested LanceModel instances in LanceDB. For example, you might have a `User` model that contains a list of `Order` models:

```python
from lancedb.pydantic import LanceModel

class Order(LanceModel):
    product: str
    quantity: int

class User(LanceModel):
    name: str
    orders: list[Order]
```

However, there is a current limitation in LanceDB: it does not yet support converting Pydantic custom types, including nested LanceModel instances, to Apache Arrow data types. This is why you might encounter a `TypeError` when trying to use `list[LanceModel]` as a field type.

## Handling Unsupported Types

When you encounter a `TypeError` with the message "Converting Pydantic type to Arrow Type: unsupported type", it means that LanceDB is unable to convert the specified Pydantic type to an Apache Arrow data type.

Currently, LanceDB supports the following type conversions:

- `int` to `pyarrow.int64`
- `float` to `pyarrow.float64`
- `bool` to `pyarrow.bool`
- `str` to `pyarrow.utf8()`
- `list` to `pyarrow.ListType`

If you need to use a Pydantic custom type, such as a nested LanceModel instance, you will need to manually convert it to a supported type.

## Troubleshooting Common Errors

Here are some solutions to common errors you might encounter when working with nested LanceModel instances in LanceDB:

- **TypeError: Converting Pydantic type to Arrow Type: unsupported type** - This error occurs when you try to use a Pydantic type that LanceDB does not support. To resolve this issue, you can manually convert the unsupported type to a supported type.

- **AttributeError: 'list' object has no attribute 'to_arrow_schema'** - This error occurs when you try to call `to_arrow_schema()` on a list of LanceModel instances. The `to_arrow_schema()` method is only available on LanceModel instances, not on lists. To resolve this issue, you can iterate over the list and call `to_arrow_schema()` on each LanceModel instance.

Remember, if you encounter an error or issue that is not covered in this guide, you can always reach out to the LanceDB community for help.