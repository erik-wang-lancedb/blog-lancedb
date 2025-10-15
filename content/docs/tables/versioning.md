---
title: "Managing Data Versioning in LanceDB"
description: "Learn how to manage, limit, or disable versioning in LanceDB to optimize storage usage, particularly in use cases where versioning is unnecessary."
weight: 5
---

LanceDB provides robust versioning capabilities, allowing you to rollback to any previous version without data duplication. However, in some use cases, particularly when inserting large amounts of data into a local filesystem database, versioning may lead to excessive storage usage. This guide will help you understand how to manage, limit, or even disable versioning in LanceDB to cater to these scenarios.

## Limiting Versioning

While LanceDB does not provide a direct way to limit versioning, you can manage the storage usage by optimizing your tables after making several small appends. This can be done using the `optimize()` method.

Here's an example in Python:

```python
import lancedb

# Connect to your LanceDB instance
db = lancedb.connect('your_instance')

# Access your table
table = db.table('your_table')

# Optimize the table
table.optimize()
```

And in TypeScript:

```typescript
import { LanceDB } from 'lancedb';

// Connect to your LanceDB instance
const db = LanceDB.connect('your_instance');

// Access your table
const table = db.table('your_table');

// Optimize the table
await table.optimize();
```

## Disabling Versioning

Currently, LanceDB does not provide an option to entirely disable versioning. However, you can manage the versions by manually deleting older versions that are no longer needed.

Here's an example in Python:

```python
import lancedb

# Connect to your LanceDB instance
db = lancedb.connect('your_instance')

# Access your table
table = db.table('your_table')

# Delete older versions
table.delete_versions(older_than='7d')
```

Please note that this will permanently delete the specified versions and they cannot be recovered.

## Troubleshooting

If you're facing issues with storage usage even after optimizing your tables or deleting older versions, it might be due to other factors such as:

- Large data insertions: If you're inserting large amounts of data at once, consider breaking it down into smaller chunks and optimizing the table after each insertion.
- Incomplete transactions: If there are any incomplete transactions, they might be holding up storage. Make sure to commit or rollback any pending transactions.

Remember, managing your data versioning effectively is crucial to optimizing your storage usage in LanceDB. If you have any further questions, feel free to reach out to our support team.