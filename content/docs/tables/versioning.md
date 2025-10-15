---
title: "Versioning & Reproducibility in LanceDB"
sidebar_title: "Versioning Tables"
description: "Learn how to implement versioning and ensure reproducibility in LanceDB. Includes version control, data snapshots, and audit trails."
weight: 3
aliases: ["/docs/concepts/tables/versioning/", "/docs/concepts/tables/versioning"]
---

LanceDB redefines data management for AI/ML workflows with built-in, automatic versioning powered by the [Lance columnar format](https://github.com/lancedb/lance). Every table mutation—appends, updates, deletions, or schema changes — is tracked with zero configuration, enabling:

- Time-Travel Debugging: Pinpoint production issues by querying historical table states.
- Atomic Rollbacks: Revert terabyte-scale datasets to any prior version in seconds.
- ML Reproducibility: Exactly reproduce training snapshots (vectors + metadata).
- Branching Workflows: Conduct A/B tests on embeddings/models via lightweight table clones.

## Basic Versioning Example

Let's create a table with sample data to demonstrate LanceDB's versioning capabilities:

### Setting Up the Table

First, let's create a table with some sample data:

{{< code language="python" >}}
import lancedb
import pandas as pd
import numpy as np
import pyarrow as pa
from sentence_transformers import SentenceTransformer

# Connect to LanceDB
db = lancedb.connect(
  uri="db://your-project-slug",
  api_key="your-api-key",
  region="us-east-1"
)

# Create a table with initial data
table_name = "quotes_versioning_example"
data = [
    {"id": 1, "author": "Richard", "quote": "Wubba Lubba Dub Dub!"},
    {"id": 2, "author": "Morty", "quote": "Rick, what's going on?"},
    {
        "id": 3,
        "author": "Richard",
        "quote": "I turned myself into a pickle, Morty!",
    },
]

# Define schema
schema = pa.schema(
    [
        pa.field("id", pa.int64()),
        pa.field("author", pa.string()),
        pa.field("quote", pa.string()),
    ]
)

table = db.create_table(table_name, data, 

## Managing Versioning and Storage in LanceDB

LanceDB provides robust versioning capabilities, allowing for fast rollbacks to any previous version without data duplication. However, in some use cases, you may want to limit or disable versioning to optimize storage usage, especially when dealing with large amounts of data. This guide will walk you through the process of managing versioning and storage in LanceDB.

### Limiting Versioning

While LanceDB does not currently provide a direct way to limit versioning, you can manage the duration of time to keep versions of the dataset by setting the `older_than` parameter. This parameter is part of the `lance.dataset.DatasetOptimizer.compact_files` method, which runs the compaction process on the table to optimize it for faster reads.

Here's a Python example:

```python
import lancedb
import datetime

# Connect to LanceDB
db = lancedb.connect("data/sample-lancedb")

# Get the table
table = db.get_table("my_table")

# Set the older_than parameter to 7 days
older_than = datetime.timedelta(days=7)

# Run the compaction process
table.compact_files(older_than=older_than)
```

In this example, versions of the dataset older than 7 days will be deleted during the compaction process.

### Disabling Versioning

To disable versioning, you can set the `older_than` parameter to a very small value, effectively deleting versions almost immediately after they are created. However, be aware that this will prevent you from rolling back to previous versions.

Here's a Python example:

```python
import lancedb
import datetime

# Connect to LanceDB
db = lancedb.connect("data/sample-lancedb")

# Get the table
table = db.get_table("my_table")

# Set the older_than parameter to 1 second
older_than = datetime.timedelta(seconds=1)

# Run the compaction process
table.compact_files(older_than=older_than)
```

In this example, versioning is effectively disabled as versions are deleted almost immediately.