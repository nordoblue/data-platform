---
title: Troubleshoot ingestion pipelines
description: Diagnose connector, discovery, schema and data-quality failures in Northstar ingestion pipelines.
product: Northstar Data Cloud
content_type: troubleshooting
audience:
  - data administrator
  - support engineer
  - software engineer
last_reviewed: 2026-10-01
tags:
  - troubleshooting
  - ingestion
  - s3
---

# Troubleshoot ingestion pipelines

Use this guide when an ingestion connector cannot connect, does not discover expected files or produces failed or incomplete runs.

Start with the symptom that matches what you see in Northstar.

## Connection test fails

A failed S3 connection test usually means that Northstar cannot assume the configured IAM role or cannot access the requested bucket prefix.

### Check the role ARN

Confirm that the connector uses the complete IAM role ARN and that the role still exists.

Example:

```text
arn:aws:iam::123456789012:role/northstar-s3-reader
```

### Check the trust policy

Confirm that the trust relationship allows the Northstar AWS principal to assume the role and includes the external ID shown in the connector setup flow.

If the external ID does not match, AWS rejects the assume-role request even when the role ARN is correct.

### Check S3 permissions

The role requires both:

- `s3:ListBucket` for the bucket
- `s3:GetObject` for the configured object prefix

If the role can list the bucket but cannot read objects, the connection test can fail after discovering a matching file.

### Check explicit denies

AWS evaluates explicit `Deny` statements before `Allow` statements. Review bucket policies, service control policies and permission boundaries if the IAM role appears correct but access is still denied.

## No files are discovered

If the connector succeeds but a run reports `0 files discovered`, compare the configured prefix and file pattern with the actual object keys.

For example, this configuration:

```text
Prefix: northstar/customer-accounts/
Pattern: customer_accounts_*.csv
```

matches:

```text
northstar/customer-accounts/customer_accounts_2026-09-30.csv
```

but does not match:

```text
northstar/customer_accounts_2026-09-30.csv
northstar/customer-accounts/accounts_2026-09-30.csv
northstar/customer-accounts/customer_accounts_2026-09-30.json
```

Also check whether the run is incremental. A previously processed object is not ingested again unless you run a full reprocessing job.

## Schema validation fails

A schema-validation failure means that the incoming source no longer conforms to the destination mapping.

Common causes include:

- A required column was removed.
- A column was renamed.
- A source value cannot be converted to the configured destination type.
- The header row is missing or malformed.
- The file delimiter changed.

### Identify the first schema change

Compare the failed object's header and representative values with the latest successful file.

Do not begin by changing the destination schema to accept anything. First determine whether the source changed intentionally or whether the export is defective.

### Type-conversion example

A destination field configured as Integer accepts:

```text
42
0
1539
```

but rejects values such as:

```text
N/A
unknown
1,539 employees
```

If text values are legitimate, change the data contract deliberately. If they are source defects, correct the producing system instead.

## Run completes with rejected rows

A `Warning` run can still write valid rows while rejecting individual records.

Open the run and review:

- Rejected row count
- Source file
- Row number
- Destination field
- Error reason
- Original value

Look for patterns rather than correcting rows one at a time. A large number of failures on the same field usually indicates a mapping or source-export problem.

### Example

If `employee_count` rejects 624 records after historically rejecting none, check whether the source recently began exporting formatted values such as `1,250` instead of `1250`.

## Dataset contains duplicate entities

If multiple rows represent the same business entity, confirm that the dataset uses the correct record key.

Run:

```sql
SELECT
  account_id,
  COUNT(*) AS occurrences
FROM customer_accounts
GROUP BY account_id
HAVING COUNT(*) > 1;
```

If this query returns rows:

1. Confirm that `account_id` is the intended business identifier.
2. Check whether whitespace or casing differences create logically duplicate keys.
3. Verify whether historical loads were configured as append-only rather than upsert.
4. Correct the record-key configuration before the next production load.

Do not delete duplicates until you understand how they were created. Otherwise the next run can recreate them.

## Dataset row count is lower than the source

A lower destination count is not automatically an ingestion failure.

Compare these metrics:

```text
rows_read
rows_written
rows_rejected
rows_updated
```

For an upsert dataset, multiple source rows can update existing records instead of increasing the final row count.

Use this SQL query to compare unique source keys with destination keys when investigating a discrepancy.

```sql
SELECT COUNT(DISTINCT account_id)
FROM customer_accounts;
```

The distinct business-key count is often a better comparison than the raw number of source lines.

## API returns `409 RUN_ALREADY_ACTIVE`

Northstar allows only one active ingestion run per dataset.

Before retrying:

1. Retrieve the active run.
2. Wait until it reaches `complete`, `warning` or `failed`.
3. Start the new run only if another synchronization is still required.

Do not retry `409` responses in a tight loop.

## API returns `429 RATE_LIMITED`

Reduce request frequency and use exponential backoff.

For example:

```text
1 second
2 seconds
4 seconds
8 seconds
16 seconds
```

Honor a `Retry-After` response header when present.

## Escalate a persistent issue

Before escalating a connector or ingestion problem, collect:

- Connector name
- Dataset ID
- Run ID
- Failure timestamp
- Source object key
- Error code and message
- Request ID for API failures
- Whether the issue reproduces on another source file
- The most recent successful run

Do not include source credentials, bearer tokens or secrets in support tickets or screenshots.
