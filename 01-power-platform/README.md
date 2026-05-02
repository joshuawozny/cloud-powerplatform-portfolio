# Power Platform & Microsoft Fabric Solutions

## 📌 Overview

This section documents Power Platform and Microsoft Fabric solutions built across enterprise and portfolio contexts. The primary project is the **NYC311 Fabric Data Platform** — an end-to-end analytics solution built on Microsoft Fabric that ingests live service request data from the NYC Open Data API, transforms it through a medallion Lakehouse architecture, and serves it through a mixed-mode Power BI semantic model with pre-materialized aggregations.

The platform demonstrates production-grade practices across the full Power Platform and Fabric stack: governed workspace configuration, notebook-orchestrated data pipelines, a DirectQuery/Dual/Import semantic model with aggregation table optimization, and a Git-integrated deployment pipeline with reviewer-gated environment promotion.

**Future additions to this section** will cover Canvas and Model-Driven Power Apps, Dataverse schema design, C# plugins, and PCF components aligned to the PL-400 certification track.

---

## 🧩 Architecture

The NYC311 platform is built entirely within a single Microsoft Fabric workspace (`NYC311-Fabric`) using a **Per User Premium (PPU)** license. A Fabric-native deployment pipeline promotes artifacts across Development, Test, and Production stages, with mandatory reviewer approval at each transition.

```
NYC Open Data 311 API
        │
        ▼
[ Fabric Pipeline: NYC311_API_Incremental ]
  ├── Notebook: Ingest_API_Live          → Bronze (raw, incremental)
  ├── Notebook: Transform_Bronze_to_Silver → Silver (cleansed, typed)
  ├── Notebook: Transform_Silver_to_Gold  → Gold (dims, fact, agg tables)
  └── Activity: Semantic Model Refresh
        │
        ▼
NYC311_Lakehouse SQL Analytics Endpoint
        │
        ▼
Power BI Semantic Model
  ├── Dimension tables  → Dual mode
  ├── Fact table        → DirectQuery
  └── Aggregation tables → Import mode
        │
        ▼
NYC311 Power BI Report
```

**Components:**
- Microsoft Fabric Lakehouse (`NYC311_Lakehouse`) with SQL analytics endpoint
- Fabric Pipeline for notebook orchestration and semantic model refresh
- PySpark/Python notebooks for medallion layer transformations
- Power BI semantic model with mixed storage modes and aggregation tables
- Fabric-native deployment pipeline (Dev / Test / Production) with reviewer gates
- GitHub repository with Git-integrated workspace and feature branch PR workflow

---

## 🛠️ Features

- Incremental API ingestion with watermark-based load tracking — no full reloads on pipeline runs
- Medallion architecture (Bronze → Silver → Gold) with Delta table storage throughout
- Gold aggregation tables materialized in the Lakehouse at pipeline execution time, surfaced as Import-mode tables in the semantic model
- Mixed storage mode semantic model delivering DirectQuery data freshness with Import-speed aggregation performance
- Fabric-native deployment pipeline with mandatory reviewer approval between Development → Test and Test → Production
- Git-integrated workspace enabling version control of all notebook, pipeline, and semantic model definitions
- Feature branch workflow with PR-gated merges triggering workspace sync

---

## 🧪 Technical Highlights

**Semantic model storage mode strategy** — Dimension tables in Dual mode allow them to serve both DirectQuery fact queries and Import-mode aggregation joins without mode conflicts. The fact table in DirectQuery ensures report data always reflects the current Lakehouse state. Pre-materialized aggregation tables in Import mode deliver sub-second performance on summary visuals without a full Import refresh of the fact table.

**Aggregation table design** — Aggregation tables are built as part of the Gold layer transformation in the Lakehouse notebook, not inside the semantic model. This keeps aggregation logic version-controlled, testable, and decoupled from the model definition. DAX Studio is used to validate aggregation hit rates and trace DirectQuery fallback by inspecting query plans and server timings against the Analysis Services endpoint.

**Tabular Editor 3** — Used for bulk DAX measure authoring, Best Practice Analyzer runs, TMDL review of serialized model definitions, and column/measure annotation management. TE3 connects to the workspace semantic model via the XMLA endpoint for live model editing independent of Power BI Desktop.

**Deployment governance** — The Fabric deployment pipeline and Git PR workflow operate as complementary controls. Git enforces code review before changes reach Development. The deployment pipeline enforces reviewer approval before changes reach Test or Production. Neither can be bypassed independently.

**Star schema design** — Single-directional relationships from fact to dimensions prevent ambiguous filter propagation. No many-to-many relationships are used; where bridge logic is needed, explicit DAX `TREATAS` patterns are used instead.

---

## 📂 Folder Structure

```
01-power-platform/
├── docs/
│   ├── fabric-environment-setup.md     ← Workspace config, PPU licensing, artifact inventory
│   ├── semantic-model-design.md        ← Storage modes, aggregations, DAX Studio, TE3
│   └── deployment-pipeline.md         ← Dev/Test/Prod pipeline, reviewer gates, Git relationship
├── screenshots/
└── README.md
```

*Canvas App, Model-Driven App, Dataverse, C# plugin, and PCF component subfolders will be added as PL-400 projects are developed.*

---

## 🚀 How to Use / Demo

To explore the platform:

1. Review the [Fabric Environment Setup](./docs/fabric-environment-setup.md) for the full workspace artifact inventory and tooling overview
2. Review the [Semantic Model Design](./docs/semantic-model-design.md) for storage mode rationale and aggregation configuration details
3. Review the [Deployment Pipeline](./docs/deployment-pipeline.md) for environment promotion and governance workflow
4. See [`03-data-analytics/`](../03-data-analytics/) for the Lakehouse architecture and the NYC311 Power BI report project files

The `.pbip` project files and TMDL semantic model definitions in `03-data-analytics/` can be opened directly in Power BI Desktop (with PBIP format enabled) and Tabular Editor 3 respectively.

---

## 🎓 Related Certifications

| Certification | Status | Relevance |
|---|---|---|
| PL-900 — Power Platform Fundamentals | ✅ 2025 | Platform foundation |
| PL-300 — Power BI Data Analyst Associate | ✅ 2026 | Semantic model, DAX, Fabric connectivity |

| PL-400 — Power Platform Developer | 📋 Planned | C# plugins, PCF components, ALM |
| DP-600 — Fabric Analytics Engineer Associate | 📋 Planned | Lakehouse, medallion architecture, Fabric pipelines |

---

## 📝 Lessons Learned

**Aggregation table matching is sensitive to schema alignment.** Column names, data types, and measure names in the Lakehouse Gold agg tables must match the semantic model's aggregation configuration exactly. Mismatches silently fall back to DirectQuery rather than raising errors — DAX Studio server timings are the only reliable way to confirm aggregation hits versus fallback.

**Dual mode is the correct choice for dimensions in a mixed-mode model.** Initially testing Import-only dimensions caused issues with aggregation queries that needed to join to DirectQuery partitions. Switching to Dual resolved the conflicts and simplified the storage mode design considerably.

**Git integration with Fabric workspace sync requires discipline around branch state.** The workspace reflects the connected branch at sync time — uncommitted local changes are not visible to the workspace. Keeping feature branches short-lived and commits frequent prevents divergence between local development state and the workspace.

---

## 🔗 Links

- [GitHub Repository](https://github.com/joshuawozny/cloud-powerplatform-portfolio)
- [03-data-analytics — NYC311 Dashboard Project](../03-data-analytics/)
- [04-devops — Pipeline & Git Workflow](../04-devops/)
- [Roadmap](../roadmap/)
