# Dataset statuses

| Status | Meaning |
| --- | --- |
| `Ready` | The dataset is available for queries and downstream workflows. |
| `Syncing` | Northstar is currently ingesting or transforming source data. |
| `Warning` | The latest sync completed with one or more non-blocking data-quality issues. |
| `Failed` | The latest sync did not complete. Open the run details to view the blocking error. |
