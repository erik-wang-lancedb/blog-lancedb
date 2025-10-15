---
title: "Managing Versioning and Directory Size in LanceDB"
description: "Learn how to control versioning and optimize directory size in LanceDB, especially when working with local filesystem databases."
weight: 8
---

{{< toc >}}

## Introduction

LanceDB is a powerful vector database that supports versioning, allowing users to rollback to any previous version without data duplication. However, when dealing with large datasets, especially on a local filesystem, this can lead to substantial storage usage. This guide will help you understand how to manage data versioning in LanceDB to optimize your storage usage.

## Controlling Versioning in LanceDB

Currently, LanceDB automatically creates new versions when you modify data through operations like update or delete. However, you might want to limit or disable this feature in certain scenarios.

### Limiting Versioning

LanceDB does not provide a direct feature to limit or omit versioning. However, you can manage the versions by manually deleting older versions that are no longer needed.

{{< code language="python" >}}
import lancedb
import os

# Connect to your LanceDB instance
db = lancedb.connect('your_database_path')

# List all versions of a table
versions = db.table('your_table').versions()

# Delete older versions
for version in versions[:-1]:  # Keep the latest version
    os.remove(version.path)
{{< /code >}}

Please note that this operation is irreversible, so make sure to backup your data if needed.

### Python SDK

To connect to your LanceDB instance using Python SDK, use the following code:

```python
import lancedb
db = lancedb.connect("./data") # Local directory for data storage
```

### TypeScript SDK

To connect to your LanceDB instance using TypeScript SDK, use the following code:

```typescript
import * as lancedb from "@lancedb/lancedb"
// Connect to local LanceDB
const db = await lancedb.connect("./data") // Local directory for data storage
```

## Optimizing Directory Size

When inserting a large amount of data in a local filesystem db, you might end up with massive LanceDB directories. After optimizing the tables, you can reduce the directory size significantly.

### Optimizing Storage

After making several small appends, you can run the compaction process on the table to optimize it for faster reads. This process reorganizes the data and can help reduce the size of your LanceDB directories.

{{< code language="python" >}}
# Run the compaction process on the table
db.table('your_table').compact_files()
{{< /code >}}

### Python SDK

To optimize the table for faster reads in Python SDK, you can use the `compact_files` function as follows:

```python
db.table.compact_files()
```

### TypeScript SDK

In TypeScript SDK, you can close the table to release any underlying resources. It's safe to call this method multiple times. Any attempt to use the table after it's closed will result in an error.

```typescript
table.close()
```

## Troubleshooting

If you're facing issues with versioning or directory size, consider the following:

- Check your filesystem: Ensure you have enough storage space and consider using a filesystem that supports large files.
- Review your data: Large datasets with complex data types can take up more storage. Consider simplifying your data or using more efficient data types.
- Contact support: If you're still having trouble, reach out to LanceDB support for assistance.