# Fabric Environment Setup — NYC311 Data Platform

## Overview

The NYC311 data platform runs on Microsoft Fabric using a **Per User Premium (PPU)** license. The environment consists of a single Fabric workspace (`NYC311-Fabric`) containing all platform artifacts — Lakehouse, notebooks, pipeline, SQL analytics endpoint, and Power BI reports. Environment promotion across Dev, Test, and Production is handled through Fabric's native deployment pipeline rather than duplicating workspaces per environment.

---

## Licensing

The platform operates on a Fabric trial / PPU license. PPU provides access to the full Fabric item catalog including Lakehouses, notebooks, pipelines, and deployment pipelines, making it suitable for development and portfolio demonstration purposes.

For production enterprise deployments, a Fabric capacity (F-SKU) or Power BI Premium capacity (P-SKU) would be used in place of PPU to support organizational-scale throughput, dedicated compute, and SLA-backed refresh guarantees.

---

## Workspace Inventory

The `NYC311-Fabric` workspace contains the following artifacts:

| Item | Type | Purpose |
|------|------|---------|
| `NYC311_Lakehouse` | Lakehouse | Primary Delta table storage for all medallion layers |
| `NYC311_Lakehouse` | SQL Analytics Endpoint | T-SQL interface for semantic model connectivity |
| `NYC311_API_Incremental` | Pipeline | Orchestrates end-to-end ingestion and refresh |
| `Ingest_API_Live` | Notebook | Incremental API ingestion to Bronze |
| `Transform_Bronze_to_Silver` | Notebook | Cleanse and conform to Silver layer |
| `Transform_Silver_to_Gold` | Notebook | Build Gold dims, fact, and aggregation tables |
| `NYC311-Report` | Power BI Report | Primary analytics report |

---

## Workspace Configuration

The workspace is configured with the following settings relevant to platform operation:

**Git integration** is enabled and connected to the `cloud-powerplatform-portfolio` GitHub repository. The workspace tracks the `dev` branch, and all item definitions (notebooks, pipeline JSON, semantic model metadata) are serialized to the repo on commit. This enables version control, code review, and automated deployment triggers.

**Deployment pipeline** is attached to the workspace as the Development stage. Promotion to Test and Production stages is managed through the pipeline — see [Deployment Pipeline](./deployment-pipeline.md) for configuration details.

**Sensitivity and endorsement** labels are not configured for this portfolio environment but would be applied in an enterprise context to meet data governance and compliance requirements.

---

## Fabric Pipeline — Orchestration Entry Point

The `NYC311_API_Incremental` pipeline is the single execution entry point for the platform. It runs three activities in sequence:

1. **Get API Data** — triggers `Ingest_API_Live` notebook, which performs incremental extraction from the NYC Open Data 311 API into the Bronze Lakehouse layer
2. **Silver to Gold** — triggers `Transform_Bronze_to_Silver` followed by `Transform_Silver_to_Gold`, producing cleansed, conformed, and aggregated Gold tables
3. **Semantic model refresh** — triggers a dataset refresh of the Power BI semantic model to update Import-mode aggregation tables and re-cache Dual-mode dimensions

The pipeline can be run manually, scheduled, or triggered programmatically. Current configuration supports manual and scheduled execution.

See [Fabric Pipeline Orchestration](../../04-devops/docs/fabric-pipeline.md) for detailed notebook and activity configuration.

---

## Tooling Outside the Fabric Service

Two external tools are used alongside the Fabric workspace for semantic model development and validation:

**DAX Studio** is used to connect directly to the semantic model's Analysis Services endpoint. Primary use cases are validating aggregation table hit rates, tracing DirectQuery fallback, and profiling query performance against the Lakehouse SQL endpoint.

**Tabular Editor 3** is used for semantic model authoring tasks beyond the Power BI Desktop surface — including bulk measure editing, TMDL review, best practice rule analysis, and preparing the model for CI/CD serialization. TE3 connects to the workspace semantic model via the XMLA endpoint, enabling live editing of the deployed model.

---

## Related Documentation

- [Semantic Model Design](./semantic-model-design.md)
- [Deployment Pipeline](./deployment-pipeline.md)
- [Lakehouse Architecture](../../03-data-analytics/docs/lakehouse-architecture.md)
- [Fabric Pipeline Orchestration](../../04-devops/docs/fabric-pipeline.md)
