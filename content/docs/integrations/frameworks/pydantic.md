---
title: "Working with Nested LanceModels in LanceDB"
description: "Guide on how to handle nested LanceModels in LanceDB and an explanation of the current limitations"
weight: 20
---

## Overview

In LanceDB, you can define your data models using the `LanceModel` class from the `lancedb.pydantic` module. This class is a custom implementation of Pydantic's `BaseModel` and it supports most of the features provided by Pydantic. However, there are some limitations when it comes to using nested `LanceModel` instances, especially when they are used within a list.

This guide will explain the current limitations, provide workarounds, and show you how to handle nested `LanceModel` instances in LanceDB.

## Current Limitations

As of LanceDB version 0.25.1, `list[LanceModel]` is not a valid field type. This means you cannot define a field in your `LanceModel` that is a list of other `LanceModel` instances. When you try to do this, you will encounter a `TypeError` indicating that the conversion of the Pydantic type to the Arrow Type is unsupported.

Here's an example of a code that will raise this error:

```python
from lancedb.pydantic import LanceModel

class SubFeature1(LanceModel):
    amount: int
    name: str

class RandomFeature1(LanceModel):
    email: str
    items: list[SubFeature1]

print(RandomFeature1.to_arrow_schema())
```

Running this code will result in the following error:

```
TypeError: Converting Pydantic type to Arrow Type: unsupported type <class '__main__.SubFeature1'>.
```

This limitation exists because LanceDB does not yet support converting Pydantic custom types, which include `LanceModel` instances. If you need this feature, consider filing a feature request on the LanceDB Github repo.

## Workarounds

While `list[LanceModel]` is not directly supported, there are workarounds that you can use to handle nested `LanceModel` instances.

### Using a Single LanceModel Instance

Instead of using a list of `LanceModel` instances, you can use a single `LanceModel` instance. This is useful when you have a one-to-one relationship between your models.

Here's an example:

```python
from lancedb.pydantic import LanceModel

class SubFeature1(LanceModel):
    amount: int
    name: str

class RandomFeature1(LanceModel):
    email: str
    item: SubFeature1

print(RandomFeature1.to_arrow_schema())
```

In this example, the `RandomFeature1` model has a `SubFeature1` instance as a field, which is supported by LanceDB.

### Using a List of Primitive Types

If you need to use a list, consider using a list of primitive types instead of a list of `LanceModel` instances. LanceDB supports lists of the following types:

- `int`
- `float`
- `bool`
- `str`

Here's an example:

```python
from lancedb.pydantic import LanceModel

class RandomFeature1(LanceModel):
    email: str
    item_amounts: list[int]

print(RandomFeature1.to_arrow_schema())
```

In this example, the `RandomFeature1` model has a list of integers as a field, which is supported by LanceDB.

## Conclusion

While the current version of LanceDB has some limitations when it comes to handling nested `LanceModel` instances, there are workarounds that you can use to achieve similar results. As LanceDB continues to evolve, more features and improvements will be added to make it easier for you to work with complex data models.