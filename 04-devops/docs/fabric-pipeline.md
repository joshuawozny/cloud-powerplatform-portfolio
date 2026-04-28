# Fabric Pipeline Orchestration — NYC311_API_Incremental

## Overview

The `NYC311_API_Incremental` Fabric pipeline is the single execution entry point for the NYC311 data platform. It orchestrates the full data flow from API ingestion through Lakehouse transformation to semantic model refresh, running three activities in a sequential dependency chain. All notebook execution occurs within the Fabric runtime — no external Azure DevOps pipeline triggers are used for data processing.

---

## Pipeline Architecture

The pipeline executes three activities in order. Each activity must succeed before the next begins. A failure at any stage halts the pipeline and leaves downstream artifacts in their last-known-good state, preserving data consistency.

```
[ Notebook: Get API Data ]
           │  success
           ▼
[ Notebook: Transform_Bronze_to_Silver ]
           │  success
           ▼
[ Notebook: Transform_Silver_to_Gold ]
           │  success
           ▼
[ Semantic Model Refresh ]
```

---

## Activity 1 — Get API Data (Ingest_API_Live Notebook)

**Purpose:** Incremental extraction from the NYC Open Data 311 API into the Bronze Lakehouse layer.

**Approach:** The notebook implements an incremental load pattern. On each execution it reads a watermark value representing the latest record loaded in the previous run, requests only records created after that watermark from the API, and appends them to the Bronze Delta table. The watermark is updated at the end of a successful run.

This approach avoids full reloads on each pipeline execution, keeping API request volume and Lakehouse write cost proportional to the volume of new data rather than the total dataset size.

**Key behaviors:**
- Reads watermark from a control table or notebook parameter
- Constructs paginated API requests for the incremental date range
- Writes raw records to the Bronze layer with an ingestion timestamp column appended
- Updates the watermark on successful completion
- On failure, the watermark is not updated — the next run will re-request the same range, ensuring no data loss

---

## Activity 2 — Transform_Bronze_to_Silver (Notebook)

**Purpose:** Cleanse and conform raw Bronze records into the Silver layer.

**Transformations applied:**
- Data type enforcement across all columns
- Null value handling with defined defaults or exclusions
- Deduplication based on unique complaint identifiers
- Column renaming to match semantic model expectations

Must succeed before Activity 3 begins. On failure, the Silver layer retains its last-known-good state and the pipeline halts, preventing partially transformed data from propagating to Gold.

---

## Activity 3 — Transform_Silver_to_Gold (Notebook)

**Purpose:** Construct Gold dimension, fact, and aggregation tables from the cleansed Silver layer.

**Transformations applied:**
- Dimension table construction with surrogate key generation
- Fact table join to dimension surrogate keys, replacing source system keys
- Aggregation table construction at defined grain combinations (date × borough, date × agency, date × complaint type, and multi-dimensional combinations)
- Delta table write using merge (upsert) logic to handle late-arriving or corrected records without full table rewrites

**Aggregation table design:** Agg tables are materialized here as pre-computed GROUP BY results over the fact grain. Building them in the Lakehouse notebook rather than inside the semantic model ensures they are version-controlled, testable, and decoupled from the model definition. Each agg table's schema is designed to match the semantic model's registered aggregation configuration — column names, data types, and measure names must align exactly for aggregation matching to function correctly.

---

## Activity 4 — Semantic Model Refresh

**Purpose:** Refresh the Power BI semantic model to update Import-mode aggregation tables and re-cache Dual-mode dimension tables following the Gold layer update.

The semantic model refresh is triggered as a native Fabric pipeline activity using the **Semantic model refresh** activity type. It targets the `NYC311` semantic model in the workspace and performs a full refresh of all Import and Dual mode partitions.

DirectQuery tables (the fact table) do not require a model refresh — they are always queried live against the Lakehouse SQL endpoint. Only the Import-mode aggregation tables and Dual-mode dimension caches need to be updated to reflect the latest Gold layer state.

The refresh completes synchronously within the pipeline — the pipeline activity waits for the refresh to succeed or fail before marking itself complete, ensuring that downstream report consumers do not see stale aggregation data after a pipeline run.

---

## Pipeline Scheduling

The pipeline supports both manual execution and scheduled runs. Current configuration for portfolio purposes uses manual execution. In a production scenario, a scheduled trigger aligned to the API data update cadence (NYC Open Data 311 data updates daily) would be configured to run the pipeline automatically.

---

## Error Handling and Monitoring

Pipeline run history is accessible from the **Activities** tab in the Fabric pipeline editor and the workspace monitoring hub. Each activity logs its start time, end time, status, and error details on failure.

Notebook-level logging is implemented within each notebook using `print` statements and structured log output, visible in the notebook run output captured by the pipeline. Watermark state ensures that a failed ingestion run does not advance the load position, making re-runs safe and idempotent.

---

## Screenshots

See the `/screenshots` folder for:
- Pipeline canvas showing the three-activity chain
- Workspace inventory showing all platform artifacts
- Example pipeline run history from the Activities tab

---

## Related Documentation

- [Lakehouse Architecture](../../03-data-analytics/docs/lakehouse-architecture.md)
- [Git Branching Strategy](./git-branching-strategy.md)
- [Deployment Pipeline](../../01-power-platform/docs/deployment-pipeline.md)
- [Fabric Environment Setup](../../01-power-platform/docs/fabric-environment-setup.md)
