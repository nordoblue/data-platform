# Platform and Data Technical Writing Sample

This repository is a portfolio sample for a fictional SaaS product named **Northstar Data Cloud**. It demonstrates documentation for a data-centric platform that supports connectors, ingestion pipelines, schema mapping, SQL validation, REST APIs, migrations and operational troubleshooting.

The sample is intentionally written as product documentation rather than as a case study. All product names, endpoints, identifiers and sample data are fictional.

## Documentation set

- [Ingest data from Amazon S3](docs/s3-data-ingestion-guide.md) — end-to-end implementation guide for configuring, validating and operating an S3 ingestion pipeline.
- [Ingestion API reference](docs/ingestion-api-reference.md) — developer reference for validating and running ingestion jobs programmatically.
- [Troubleshoot ingestion pipelines](docs/troubleshooting-ingestion.md) — symptom-based troubleshooting for common connector, schema and data-quality failures.

## What this sample demonstrates

- Technical product documentation for a complex, data-centric SaaS platform
- Clear guidance for both technical and less-technical readers
- Structured content using front matter, reusable terminology and predictable information patterns
- Data connector and ingestion concepts
- ETL/ELT and schema-mapping concepts
- SQL and relational data validation
- REST API documentation and test examples
- Migration guidance
- System diagrams and task-oriented tutorials
- Troubleshooting and error recovery
- Security and permission considerations
- Cross-linking between conceptual, task and reference content

## Repository structure

```text
.
├── README.md
├── config/
│   └── content-variables.yml
├── docs/
│   ├── ingestion-api-reference.md
│   ├── s3-data-ingestion-guide.md
│   └── troubleshooting-ingestion.md
└── snippets/
    └── dataset-statuses.md
```

## Editorial approach

The documentation follows a task-oriented structure:

1. State the user goal.
2. Define prerequisites and permissions.
3. Explain only the architecture needed to complete the task.
4. Provide numbered procedures with observable outcomes.
5. Include validation steps so readers can confirm success.
6. Separate troubleshooting from the happy path while linking to it at the point of need.
7. Use reference material for API behavior that would otherwise interrupt the tutorial.

The content is written to be scannable in a help center, reusable in training material and understandable to readers with different levels of technical depth.
