---
title: Ingestion API reference
description: Run ingestion jobs and retrieve their status through the Northstar REST API.
product: Northstar Data Cloud
content_type: api-reference
audience:
  - software engineer
  - data engineer
last_reviewed: 2026-10-01
tags:
  - api
  - ingestion
  - rest
---

# Ingestion API reference

Use the Ingestion API to start dataset synchronization and retrieve the status of ingestion runs.

Base URL:

```text
https://api.example.com/v1
```

## Authentication

Send a bearer token in the `Authorization` header.

```http
Authorization: Bearer <token>
```

Do not place API tokens in query parameters, source control or client-side application code.

## Start an ingestion run

`POST /datasets/{dataset_id}/ingestion-runs`

Starts an ingestion run for an existing dataset.

### Path parameters

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `dataset_id` | String | Yes | Unique identifier of the dataset to synchronize. |

### Request

```bash
curl --request POST \
  --url https://api.example.com/v1/datasets/ds_customer_accounts/ingestion-runs \
  --header "Authorization: Bearer $NORTHSTAR_API_TOKEN" \
  --header "Content-Type: application/json" \
  --data '{
    "mode": "incremental"
  }'
```

### Request body

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `mode` | String | No | `incremental` processes new or changed source objects. `full` reprocesses the configured source scope. Default: `incremental`. |

### `202 Accepted`

```json
{
  "id": "run_01J8QX7H2K4W",
  "dataset_id": "ds_customer_accounts",
  "status": "queued",
  "mode": "incremental",
  "created_at": "2026-10-01T02:00:01Z"
}
```

A `202` response means that Northstar accepted the request. It does not mean that ingestion has completed.

Use the returned run ID to retrieve the final status.

## Retrieve an ingestion run

`GET /ingestion-runs/{run_id}`

Returns current status and processing metrics for one run.

### Request

```bash
curl --request GET \
  --url https://api.example.com/v1/ingestion-runs/run_01J8QX7H2K4W \
  --header "Authorization: Bearer $NORTHSTAR_API_TOKEN" \
  --header "Accept: application/json"
```

### `200 OK`

```json
{
  "id": "run_01J8QX7H2K4W",
  "dataset_id": "ds_customer_accounts",
  "status": "complete",
  "started_at": "2026-10-01T02:00:03Z",
  "completed_at": "2026-10-01T02:02:41Z",
  "files_discovered": 1,
  "rows_read": 12481,
  "rows_written": 12481,
  "rows_rejected": 0,
  "rows_updated": 417
}
```

### Run statuses

| Status | Description |
| --- | --- |
| `queued` | The run is waiting for processing capacity. |
| `running` | Northstar is reading, validating or writing data. |
| `complete` | The run completed without blocking errors. |
| `warning` | The run completed but one or more rows were rejected or another non-blocking issue occurred. |
| `failed` | Processing stopped because of a blocking error. |

## Error responses

Error responses use a consistent envelope.

```json
{
  "error": {
    "code": "DATASET_NOT_FOUND",
    "message": "Dataset ds_customer_account was not found.",
    "request_id": "req_01J8R15MN6P"
  }
}
```

### Common errors

| HTTP status | Code | Meaning | Resolution |
| --- | --- | --- | --- |
| `400` | `INVALID_RUN_MODE` | `mode` contains an unsupported value. | Use `incremental` or `full`. |
| `401` | `UNAUTHENTICATED` | Token is missing, invalid or expired. | Send a valid bearer token. |
| `403` | `INSUFFICIENT_PERMISSION` | Token cannot run ingestion for the dataset. | Add the required dataset permission or use another token. |
| `404` | `DATASET_NOT_FOUND` | The dataset ID does not exist or is not visible to the caller. | Verify the ID and workspace. |
| `409` | `RUN_ALREADY_ACTIVE` | Another run is already processing the dataset. | Wait for the active run to complete before retrying. |
| `429` | `RATE_LIMITED` | Request rate exceeded the API limit. | Retry with exponential backoff. |
| `500` | `INTERNAL_ERROR` | Northstar could not complete the request. | Retry if appropriate and provide the `request_id` to support if the error persists. |

## Poll until a run completes

Do not poll continuously. Start with a short delay and increase the interval for longer-running jobs.

Example pseudocode:

```python
terminal_states = {"complete", "warning", "failed"}

while True:
    run = get_ingestion_run(run_id)

    if run["status"] in terminal_states:
        break

    sleep(10)
```

Treat both `warning` and `failed` as states that require review before downstream automation continues.

## Idempotency and retries

A network timeout can occur after the server accepts a `POST` request but before the client receives the response. If your integration automatically retries ingestion requests, include an idempotency key so the retry does not start a duplicate run.

```http
Idempotency-Key: 3fe9c7b0-54a9-4b11-8fe1-3dbb25827f81
```

Reuse the same key only when retrying the same logical request.
