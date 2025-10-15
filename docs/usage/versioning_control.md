---
title: "Managing Versioning and Optimizing Storage in LanceDB"
description: "Learn how to control versioning and optimize storage in LanceDB to manage large directory sizes."
weight: 4
---

{{< toc >}}

## Introduction

LanceDB provides robust versioning capabilities that allow for fast rollbacks to any previous version without data duplication. However, in certain use cases, especially when dealing with large amounts of data stored in a local filesystem database, versioning can lead to large directory sizes. This guide will provide you with the necessary steps to manage versioning and optimize storage in LanceDB.

## Limiting Versioning in LanceDB

While LanceDB does not currently have a built-in feature to limit or disable versioning, you can manage versioning by controlling the duration of time to keep versions of the dataset.

Here's how you can do it in Python:

{{< code language="python" >}}
import lancedb
from datetime import timedelta

# Connect to your LanceDB instance
db = lancedb.connect('your_database_uri')

# Specify the duration to keep versions of the dataset
duration = timedelta(days=7)  # keep versions for 7 days

# Apply the duration to your table
table = db.table('your_table_name')
table.set_version_retention(duration)
{{< /code >}}

And here's the equivalent in TypeScript:

{{< code language="typescript" >}}
import { LanceDB } from 'lancedb';

// Connect to your LanceDB instance
const db = new LanceDB('your_database_uri');

// Specify the duration to keep versions of the dataset
const duration = 7 * 24 * 60 * 60 * 1000;  // keep versions for 7 days

// Apply the duration to your table
const table = db.table('your_table_name');
table.setVersionRetention(duration);
{{< /code >}}

## Optimizing Storage in LanceDB

After inserting a large amount of data, you can optimize your tables to reduce the size of your LanceDB directories. Here's how to do it in Python:

{{< code language="python" >}}
import lancedb

# Connect to your LanceDB instance
db = lancedb.connect('your_database_uri')

# Optimize your table
table = db.table('your_table_name')
table.optimize()
{{< /code >}}

And here's the equivalent in TypeScript:

{{< code language="typescript" >}}
import { LanceDB } from 'lancedb';

// Connect to your LanceDB instance
const db = new LanceDB('your_database_uri');

// Optimize your table
const table = db.table('your_table_name');
table.optimize();
{{< /code >}}

## Troubleshooting

If you're still experiencing large directory sizes after limiting versioning and optimizing your tables, please ensure that there are no in-progress transactions that might be preventing old versions from being deleted. If the problem persists, please contact us at support@lancedb.com.

## Conclusion

While LanceDB's versioning feature is designed to provide robust data protection, it can lead to large directory sizes when dealing with large amounts of data. By controlling the duration of time to keep versions of the dataset and regularly optimizing your tables, you can effectively manage your storage in LanceDB.