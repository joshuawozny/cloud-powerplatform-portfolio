# Lakehouse Architecture — NYC311 Data Platform

## Overview

The NYC311 data platform is built on a single Microsoft Fabric Lakehouse (`NYC311_Lakehouse`) shared across the Dev, Test, and Production deployment stages. All data ingestion, transformation, and serving layers operate within this Lakehouse, with environment promotion managed through Fabric's native deployment pipeline rather than separate Lakehouse instances.

The platform follows a **Medallion architecture** — Bronze, Silver, and Gold — where each layer represents an increasing level of data quality, structure, and business readiness.

---

## Medallion Layer Design

### Bronze — Raw Ingestion

Bronze is the landing zone for raw API data. Records are written as-is from the NYC Open Data 311 API with minimal transformation — no type casting, no deduplication, no filtering. The goal is fidelity and auditability: if a question arises about source data quality, Bronze is the reference point.

Ingestion is incremental. The `Ingest_API_Live` notebook tracks the last loaded record and requests only new data on each pipeline run, keeping API call volume and Lakehouse write costs manageable.

### Silver — Cleansed and Conformed

The `Transform_Bronze_to_Silver` notebook promotes Bronze records into a cleansed, typed, and deduplicated Silver layer. Key transformations applied at this stage include:

- Data type enforcement (timestamps, categoricals, numeric fields)
- Null handling and default value assignment
- Duplicate record resolution based on unique complaint keys
- Structural normalization to align column naming with the semantic model's expectations

Silver is the integration layer — it is where data from disparate API responses is made consistent and queryable, but it is not yet shaped for analytics consumption.

### Gold — Analytics-Ready

The `Transform_Silver_to_Gold` notebook produces the final analytics layer. Gold contains dimension tables, a central fact table, and aggregation tables, all persisted as Delta tables in the Lakehouse. This layer is the direct source for the Power BI semantic model via the Lakehouse SQL analytics endpoint.

---

## Gold Layer Table Design

### Dimension Tables

Dimension tables are built from conformed Silver attributes — complaint type, agency, borough, location, and date. Each dimension carries a surrogate key generated at the Gold layer, with the source system key retained as an alternate key for traceability.

Dimensions are consumed by the semantic model in **Dual storage mode**, meaning they are cached in the semantic model's in-memory store while remaining queryable via DirectQuery against the Lakehouse SQL endpoint. This allows dimensions to serve both Import-mode aggregation queries and DirectQuery fact queries without mode conflicts.

### Fact Table

The central fact table (`fact_complaints`) holds complaint-level grain records joined to surrogate keys from each dimension. It is consumed by the semantic model in **DirectQuery mode**, which keeps the semantic model lightweight and ensures report queries always reflect the current state of the Lakehouse without requiring a scheduled dataset refresh.

The DirectQuery connection is made through the Lakehouse SQL analytics endpoint, which exposes Gold Delta tables as a T-SQL queryable interface.

### Aggregation Tables

Aggregation tables are created as part of the Gold layer construction in the `Transform_Silver_to_Gold` notebook. Rather than building aggregations inside the semantic model, they are materialized as pre-aggregated Delta tables in the Lakehouse and surfaced to the semantic model as **Import mode** tables.

This approach offers several advantages: aggregations are computed once during pipeline execution rather than at query time, Import-mode caching delivers sub-second response on high-grain summary visuals, and the aggregation logic is version-controlled alongside the rest of the transformation code rather than embedded invisibly in the semantic model.

The semantic model's aggregation configuration maps these Import tables as alternatives to the DirectQuery fact for defined measure/grouping combinations. DAX Studio is used to validate aggregation hit rates and diagnose fallback-to-DirectQuery scenarios.

---

## SQL Analytics Endpoint

The `NYC311_Lakehouse` SQL analytics endpoint is automatically provisioned by Fabric alongside the Lakehouse. It exposes all Gold Delta tables as a read-only T-SQL interface without requiring a separate database or data warehouse.

The semantic model connects to Gold exclusively through this endpoint. No direct Delta table file paths are used in the model — all queries are routed through the SQL endpoint to ensure consistent query folding behavior and compatibility with DirectQuery mode.

---

## Data Flow Summary

```
NYC Open Data 311 API
        │
        ▼
[ Bronze ] ── Ingest_API_Live (incremental, raw)
        │
        ▼
[ Silver ] ── Transform_Bronze_to_Silver (cleanse, type, deduplicate)
        │
        ▼
[ Gold   ] ── Transform_Silver_to_Gold (dims, fact, agg tables)
        │
        ▼
NYC311_Lakehouse SQL Analytics Endpoint
        │
        ▼
Power BI Semantic Model (Dual dims / DQ fact / Import aggs)
        │
        ▼
NYC311 Power BI Report
```

---

## Related Documentation

- [Fabric Pipeline Orchestration](../../04-devops/docs/fabric-pipeline.md)
- [Semantic Model Design](../../01-power-platform/docs/semantic-model-design.md)
- [Deployment Pipeline](../../01-power-platform/docs/deployment-pipeline.md)
