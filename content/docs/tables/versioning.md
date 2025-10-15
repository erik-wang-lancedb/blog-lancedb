---
title: "Managing Versioning and Directory Size in LanceDB"
description: "Learn how to control versioning and optimize directory size in LanceDB, especially when working with local filesystem databases."
weight: 8
---

{{< toc >}}

## Introduction

LanceDB supports versioning, allowing you to roll back to any previous version without data duplication. However, in some use cases, you might want to limit or entirely omit versioning. This guide will help you understand how to manage versioning and directory size in LanceDB, particularly when working with local filesystem databases.

## Controlling Versioning in LanceDB

Currently, LanceDB automatically creates new versions when you modify data through operations like update or delete. However, you might want to limit or disable this feature in certain scenarios.

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

If you're facing issues with versioning or directory size, make sure you're using the latest version of LanceDB. If the problem persists, please contact our support team.

## Conclusion

Understanding how to manage versioning and directory size in LanceDB can help you optimize your data storage and retrieval processes. While LanceDB offers automatic versioning, you can control this feature based on your specific needs. Similarly, you can optimize your directory size to ensure efficient data storage.