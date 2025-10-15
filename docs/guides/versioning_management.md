---
title: "Managing Versioning and Storage in LanceDB"
description: "Learn how to control versioning and optimize storage space in LanceDB."
weight: 5
---

LanceDB provides robust versioning capabilities, allowing for fast rollbacks to any previous version without data duplication. However, in some use cases, you might want to limit or even disable versioning to optimize storage space. This guide will walk you through how to manage versioning and storage in LanceDB.

## Limiting Versioning

While LanceDB does not currently provide a direct way to limit versioning, you can manage the duration of time to keep versions of the dataset by setting the `older_than` parameter.

Here's a Python example:

```python
from datetime import timedelta

# Set the duration to keep versions
older_than = timedelta(days=7)

# Connect to LanceDB
db = lancedb.connect(uri)

# Set the older_than parameter
db.set_older_than(older_than)
```

And here's a TypeScript example:

```typescript
import { Duration } from 'luxon';

// Set the duration to keep versions
const older_than = Duration.fromObject({ days: 7 });

// Connect to LanceDB
const db = lancedb.connect(uri);

// Set the older_than parameter
db.setOlderThan(older_than);
```

## Disabling Versioning

To disable versioning entirely, you can set the `older_than` parameter to a very small duration. Note that this will effectively delete all previous versions of your data, so use this option with caution.

Here's a Python example:

```python
from datetime import timedelta

# Set the duration to keep versions
older_than = timedelta(seconds=1)

# Connect to LanceDB
db = lancedb.connect(uri)

# Set the older_than parameter
db.set_older_than(older_than)
```

And here's a TypeScript example:

```typescript
import { Duration } from 'luxon';

// Set the duration to keep versions
const older_than = Duration.fromObject({ seconds: 1 });

// Connect to LanceDB
const db = lancedb.connect(uri);

// Set the older_than parameter
db.setOlderThan(older_than);
```

## Optimizing Storage Space

After making several small appends, you can run the compaction process on the table to optimize it for faster reads. This can be particularly useful when dealing with large amounts of data.

Here's a Python example:

```python
# Connect to LanceDB
db = lancedb.connect(uri)

# Get the table
table = db.get_table('my_table')

# Run the compaction process
table.compact_files()
```

And here's a TypeScript example:

```typescript
// Connect to LanceDB
const db = lancedb.connect(uri);

// Get the table
const table = db.getTable('myTable');

// Run the compaction process
table.compactFiles();
```

## Troubleshooting

If you encounter issues when trying to limit or disable versioning, or when optimizing storage space, please check the following:

- Ensure you are using the latest version of LanceDB.
- Make sure you are connected to LanceDB before trying to set the `older_than` parameter or run the compaction process.
- If you are trying to disable versioning, remember that this will delete all previous versions of your data. Make sure this is what you want before proceeding.

If you continue to experience issues, please reach out to our support team for further assistance.