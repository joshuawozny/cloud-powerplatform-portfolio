# Semantic Model Design — NYC311 Data Platform

## Overview

The NYC311 semantic model connects to Gold layer tables in the `NYC311_Lakehouse` SQL analytics endpoint and uses a mixed storage mode strategy to balance query performance, data freshness, and report responsiveness. The model is authored and maintained using Power BI Desktop and Tabular Editor 3, with DAX Studio used for performance validation and aggregation diagnostics.

---

## Storage Mode Strategy

The model uses three storage modes applied intentionally across different table types. The combination allows DirectQuery queries against live Lakehouse data while delivering Import-speed performance for the majority of dashboard interactions.

### Dimension Tables — Dual Mode

All dimension tables (complaint type, agency, borough, location, date) are configured in **Dual mode**. Dual mode tables are cached in the model's in-memory store and simultaneously remain queryable via DirectQuery against the Lakehouse SQL endpoint.

This dual availability is essential for the aggregation architecture. When a query hits an Import-mode aggregation table, the engine needs to join it with dimension values — Dual mode ensures those joins resolve in-memory rather than generating a Lakehouse round-trip. If the aggregation is not available for a given query, Dual mode dimensions fall back to DirectQuery seamlessly without mode conflicts.

### Fact Table — DirectQuery

The central fact table (`fact_complaints`) is configured in **DirectQuery mode**. All queries against complaint-level grain data are sent live to the Lakehouse SQL analytics endpoint, ensuring reports always reflect the current state of the Gold layer without requiring an Import refresh cycle.

DirectQuery is appropriate here because the fact table is large, refreshes incrementally, and does not need to be fully resident in memory — the aggregation layer handles the performance-sensitive summary queries.

### Aggregation Tables — Import Mode

Pre-aggregated tables materialized in the Gold Lakehouse layer are surfaced in the semantic model as **Import mode** tables. These tables are refreshed on each pipeline execution, when the final pipeline stage triggers a semantic model refresh.

Import mode delivers in-memory query performance for high-traffic summary visuals — borough-level complaint counts, agency response distributions, time-series trend lines — without hitting the Lakehouse SQL endpoint at query time. The aggregation configuration in the model maps these Import tables as preferred alternatives to the DirectQuery fact for defined measure and grouping combinations.

---

## Aggregation Configuration

Aggregation tables are registered in the semantic model's aggregation configuration to define which column groupings and measures they can serve. When a DAX query's GROUP BY columns and measures match an aggregation table's registered configuration, the engine routes the query to the Import-mode agg table rather than the DirectQuery fact.

Key configuration decisions:

- Aggregation tables cover the most common report grain combinations — date, borough, agency, complaint type, and combinations thereof
- Measures that cannot be aggregated from pre-computed subtotals (e.g., distinct count of unique requesters) are excluded from aggregation coverage and always fall back to DirectQuery
- The aggregation precedence order is defined explicitly to avoid ambiguous matches across multiple agg tables

---

## DAX Studio — Aggregation Validation

DAX Studio is used to validate that the aggregation configuration is working as intended. The primary diagnostic workflow is:

1. Connect DAX Studio to the workspace semantic model via the Analysis Services endpoint
2. Enable **Query Plan** and **Server Timings** traces
3. Run representative DAX queries matching report visuals
4. Inspect the query plan for `VertiPaq SE` (aggregation hit) vs. `DirectQuery` (fallback) engine calls
5. Where unexpected fallback occurs, inspect the aggregation mapping for missing column registrations or incompatible measure definitions

Common fallback causes diagnosed in this project include grouping by columns not registered in the agg table configuration and using measures with CALCULATE filters that prevent agg matching.

---

## Tabular Editor 3 — Model Authoring

Tabular Editor 3 connects to the semantic model via the workspace XMLA endpoint and is used for tasks beyond the Power BI Desktop surface:

**Bulk measure management** — measures are authored and reviewed in TE3's DAX editor, which provides syntax highlighting, auto-complete, and inline error detection superior to the Desktop measure dialog.

**Best practice analysis** — TE3's Best Practice Analyzer is run against the model to flag issues including measures lacking format strings, columns with high cardinality left visible to the report layer, and relationships with ambiguous cross-filter directions.

**TMDL review** — the model's Tabular Model Definition Language representation is reviewed in TE3 to verify that Git-serialized model changes are complete and correct before promotion through the deployment pipeline.

**Model documentation** — measure descriptions and column annotations are maintained in TE3 and serialized to the model definition, making them visible to report authors in Power BI Desktop's field list.

---

## Relationship Design

The model follows a **star schema** with the fact table at the center and dimension tables on the outer ring. All relationships are single-directional (fact to dimension) to prevent ambiguous filter propagation. No many-to-many relationships are used — where bridge tables would be required, the model uses explicit DAX TREATAS patterns instead to maintain predictable filter behavior.

---

## Related Documentation

- [Lakehouse Architecture](../../03-data-analytics/docs/lakehouse-architecture.md)
- [Fabric Environment Setup](./fabric-environment-setup.md)
- [Deployment Pipeline](./deployment-pipeline.md)
