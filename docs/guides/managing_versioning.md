---
title: "Managing Versioning and Storage in LanceDB"
description: "Learn how to limit or disable versioning and optimize storage in LanceDB."
weight: 5
---

# Managing Versioning and Storage in LanceDB

LanceDB is designed to handle large amounts of data efficiently. However, when inserting a large amount of data, you might end up with massive LanceDB directories. This guide will help you understand how to manage versioning and storage in LanceDB.

## Limiting or Disabling Versioning

By default, LanceDB creates new versions of your data when you modify it. This feature allows for fast rollbacks to any previous version without data duplication. However, in some use cases, you might want to limit or disable versioning.

While there's currently no built-in feature to limit or disable versioning, you can manage this manually by compacting your tables after making several small appends. This will optimize the table for faster reads and reduce the storage size.

Here's an example of how you can do this in Python:

{{< code language="python" >}}
import lancedb

# Connect to your LanceDB instance
db = lancedb.connect("data/sample-lancedb")

# Create a table with sample data
data = [
    {"vector": row, "item": f"item {i}"}
    for i, row in enumerate(np.random.random((10_000, 1536)).astype("float32"))
]
tbl = db.create_table("vector_search", data=data)

# Compact the table
tbl.compact_files()
{{< /code >}}

## Optimizing Storage After Data Insertion

If you're dealing with large data insertions, it's important to optimize your storage to prevent excessive usage. You can do this by running the compaction process on your tables.

Here's an example of how you can do this in Python:

{{< code language="python" >}}
import lancedb

# Connect to your LanceDB instance
db = lancedb.connect("data/sample-lancedb")

# Create a table with sample data
data = [
    {"vector": row, "item": f"item {i}"}
    for i, row in enumerate(np.random.random((10_000, 1536)).astype("float32"))
]
tbl = db.create_table("vector_search", data=data)

# Run the compaction process on the table
tbl.compact_files()
{{< /code >}}

## Troubleshooting

If you're still experiencing issues with storage size after following these steps, make sure that you're not running any in-progress transactions that might be preventing files from being deleted.

Remember, LanceDB is designed to handle large amounts of data efficiently. If you're regularly performing operations on large datasets, consider reaching out to our support team for further assistance.

We hope this guide helps you manage versioning and storage in LanceDB more effectively. If you have any questions or need further assistance, don't hesitate to reach out to our support team.