# DevOps — Pipeline Orchestration & Source Control

## 📌 Overview

This section documents the DevOps practices and pipeline architecture for the NYC311 Fabric Data Platform. The current implementation uses **Microsoft Fabric-native pipeline orchestration** for data engineering automation and **GitHub with a feature branch workflow** for source control, code review, and deployment gating — without a separate Azure DevOps service.

The platform's DevOps posture is built around two complementary controls: Git enforces that no unreviewed code reaches the Development workspace, and the Fabric deployment pipeline enforces reviewer approval before changes reach Test or Production. Together they provide a layered governance model appropriate for a production data platform.

**Future additions to this section** will introduce Azure DevOps YAML pipelines, Power Platform solution export/import automation, and Infrastructure as Code (Bicep/Terraform) aligned to the AZ-400 and PL-400 certification tracks.

---

## 🧩 Architecture

### Pipeline Orchestration

The `NYC311_API_Incremental` Fabric pipeline is the single execution entry point for the data platform. It runs four activities in a sequential dependency chain — each stage must succeed before the next begins.

```
[ Notebook: Ingest_API_Live ]
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

### Deployment Flow

```
feature branch
      │  PR review + approval required
      ▼
    main ──► Fabric workspace sync (Development)
                    │  reviewer approval required
                    ▼
                  Test
                    │  reviewer approval required
                    ▼
                Production
```

All Fabric artifact definitions — notebooks, pipeline JSON, semantic model TMDL, report definitions — are version-controlled in GitHub via the Fabric workspace Git integration. Merges to `main` trigger a workspace sync, keeping the Development stage current with the latest approved code.

---

## 🛠️ Pipeline Features

- **Incremental ingestion** — watermark-based load tracking ensures only new API records are requested on each run; no full reloads
- **Sequential activity dependencies** — pipeline halts on any notebook failure, preserving last-known-good state in downstream layers
- **Semantic model refresh as a pipeline activity** — Import-mode aggregation tables and Dual-mode dimension caches are refreshed synchronously at the end of each run, so report consumers never see stale aggregation data after a pipeline execution
- **Idempotent runs** — the watermark is only advanced on successful ingestion; a failed run re-processes the same range on retry with no data loss or duplication
- **Reviewer-gated deployment** — mandatory human approval at Development → Test and Test → Production transitions enforced at the Fabric deployment pipeline level
- **PR-gated merges** — at least one reviewer approval required before any feature branch can merge to `main`

---

## 🧪 Source Control & Branching

Development uses a **feature branch workflow** managed through GitHub, with VS Code as the local Git client for staging, committing, and PR management.

**Branch conventions:**
```
feature/incremental-api-ingestion
feature/gold-aggregation-tables
feature/semantic-model-storage-modes
fix/silver-null-handling
docs/lakehouse-architecture
```

**Serialized artifact types under version control:**

| Artifact | Format |
|----------|--------|
| Fabric pipeline | JSON activity definitions |
| Notebooks | `.py` source files |
| Semantic model | TMDL (Tabular Model Definition Language) |
| Power BI reports | Report definition JSON |

The Lakehouse schema is defined implicitly through notebook transformation logic — no separate schema migration files are maintained. The Gold layer table structure is an output of the `Transform_Silver_to_Gold` notebook and is therefore version-controlled alongside it.

---

## 🏗️ Infrastructure as Code

IaC is not yet implemented in this portfolio. Bicep and/or Terraform modules for Fabric workspace provisioning, capacity configuration, and Azure resource deployment are planned as part of the **AZ-104** and **AZ-400** certification tracks.

When added, IaC content will cover:
- Fabric workspace and capacity provisioning
- Azure resource group and service principal configuration
- Parameterized environment definitions for Dev/Test/Production

---

## 📂 Folder Structure

```
04-devops/
├── docs/
│   ├── fabric-pipeline.md          ← Pipeline activity details, incremental pattern, error handling
│   └── git-branching-strategy.md   ← Branch workflow, PR requirements, VS Code, artifact serialization
├── screenshots/
└── README.md
```

*Azure DevOps YAML pipeline definitions and IaC modules will be added as AZ-400 and PL-400 projects are developed.*

---

## 🎓 Related Certifications

| Certification | Status | Relevance |
|---|---|---|
| PL-300 — Power BI Data Analyst Associate | 🎯 In Progress | Semantic model refresh, pipeline-triggered dataset updates |
| PL-400 — Power Platform Developer | 📋 Planned | Power Platform ALM, solution export/import automation |
| AZ-104 — Azure Administrator Associate | 📋 Planned | Azure resource management, service principals, IaC fundamentals |
| AZ-400 — Azure DevOps Engineer Expert | 📋 Planned | YAML pipelines, release management, deployment automation |

---

## 📝 Lessons Learned

**Fabric-native pipelines are sufficient for data engineering orchestration at this scale.** A separate Azure DevOps pipeline for notebook triggering would add complexity without meaningful benefit for a single-workspace, single-team platform. The Fabric pipeline activity model covers sequential dependencies, error handling, and semantic model refresh natively.

**Watermark management requires explicit failure handling.** Early pipeline iterations advanced the watermark before confirming the write had fully committed, resulting in skipped records on retry. Moving the watermark update to the final step of a successful notebook run resolved this. The pattern — read watermark, process, write, then update watermark — is now consistent across all incremental notebooks.

**TMDL serialization makes semantic model changes reviewable in PRs.** Before enabling TMDL format, semantic model changes were opaque binary diffs. TMDL surfaces measure definitions, relationship configurations, and storage mode settings as readable text, making PR review of model changes substantive rather than ceremonial.

---

## 🔗 Links

- [GitHub Repository](https://github.com/joshuawozny/cloud-powerplatform-portfolio)
- [01-power-platform — Fabric Environment & Deployment Pipeline](../01-power-platform/)
- [03-data-analytics — Lakehouse Architecture & NYC311 Project](../03-data-analytics/)
- [Roadmap](../roadmap/)
