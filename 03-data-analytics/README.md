# 📊 Data Analytics Portfolio

This section showcases projects focused on data engineering, semantic modeling, DAX development, and analytical visualization using Microsoft Fabric and Power BI. Projects reflect both professional experience delivering production analytics platforms — including work at IncWorx Consulting and the Vermont Agency of Transportation — and ongoing development across the PL-300 and DP-600 certification tracks.

The approach throughout emphasizes clean, well-structured data models, governed data pipelines, meaningful DAX measures, and professional documentation that mirrors enterprise engineering standards.

---

## 📂 Projects in This Section

### 1. NYC Open Data — Fabric Data Platform & Power BI Dashboard ✅

A full end-to-end data platform built on Microsoft Fabric using the NYC Open Data 311 Service Requests API. The project covers the complete data engineering and analytics lifecycle — from incremental API ingestion through a medallion Lakehouse architecture to a mixed-mode Power BI semantic model with pre-materialized aggregation tables.

**What it demonstrates:**
- Medallion architecture (Bronze / Silver / Gold) using Delta tables in a Fabric Lakehouse
- Fabric pipeline orchestration across four sequential notebook and refresh activities
- Mixed storage mode semantic model — Dual mode dimensions, DirectQuery fact, Import aggregations
- Aggregation table design and validation using DAX Studio
- Semantic model authoring and best practice analysis with Tabular Editor 3
- Git-integrated deployment pipeline with Dev / Test / Production stages and reviewer gates

➡️ [`./powerbi-nyc311`](./powerbi-nyc311/)

---

### 2. (Future) Time Intelligence DAX Library

A reusable library of common and advanced DAX patterns covering time intelligence, running totals, period-over-period comparisons, and ranking. Intended as both a reference and a demonstration of DAX fluency beyond template measures.

➡️ `./dax-library` *(planned)*

---

### 3. (Future) Data Model Optimization Case Study

A before-and-after comparison of a poorly structured import model versus a star schema with aggregation tables. Will document the diagnostic process, modeling decisions, and measurable performance impact — directly aligned to PL-300 and DP-600 exam objectives.

➡️ `./model-optimization` *(planned)*

---

## 🧩 Platform Architecture (NYC311)

```
NYC Open Data 311 API
        │  incremental ingestion
        ▼
[ Bronze ] Raw Delta tables — Fabric Lakehouse
        │
        ▼
[ Silver ] Cleansed, typed, deduplicated
        │
        ▼
[ Gold  ] Dims + Fact + Aggregation tables
        │
        ▼
SQL Analytics Endpoint → Power BI Semantic Model
  ├── Dimension tables  (Dual mode)
  ├── Fact table        (DirectQuery)
  └── Aggregation tables (Import)
        │
        ▼
NYC311 Power BI Report
```

Full architecture documentation: [`./docs/lakehouse-architecture.md`](./docs/lakehouse-architecture.md)

---

## 🎓 Certifications Supported

| Certification | Status | Project Alignment |
|---|---|---|
| PL-900 — Power Platform Fundamentals | ✅ 2025 | Platform foundation |
| PL-300 — Power BI Data Analyst Associate | 🎯 Active | NYC311 semantic model, DAX, visualization |
| PL-200 — Power Platform Functional Consultant | 📋 Planned | Future Canvas App / Dataverse projects |
| DP-600 — Fabric Analytics Engineer Associate | 📋 Planned | NYC311 Lakehouse, medallion architecture, pipelines |

---

## 🧠 Skills Demonstrated

**Data Engineering**
- Medallion Lakehouse architecture with Delta table storage
- Fabric pipeline orchestration and notebook-based transformations
- Incremental API ingestion with watermark-based load tracking
- Bronze / Silver / Gold layer design and upsert logic

**Semantic Modeling**
- Star schema design with single-directional relationships
- Mixed storage mode strategy (Dual / DirectQuery / Import)
- Aggregation table configuration and DAX Studio validation
- Tabular Editor 3 for bulk authoring, BPA, and TMDL review

**DAX & Analytics**
- Time intelligence (YTD, YoY, period comparisons)
- KPI measures and ranking patterns
- Aggregation-aware measure design
- Performance-conscious DAX patterns

**Visualization**
- KPI cards, trend lines, category breakdowns, map visuals
- Drill-through pages and tooltip pages
- Role-based access and report-layer governance

**DevOps & Documentation**
- Git-integrated Fabric workspace with feature branch PR workflow
- Architecture documentation, data dictionaries, DAX measure libraries
- Lessons learned and reproducibility documentation

---

## 📂 Section Structure

```
03-data-analytics/
├── docs/
│   └── lakehouse-architecture.md       ← Medallion design, Gold table strategy, SQL endpoint
├── powerbi-nyc311/                     ← Primary project (complete)
│   ├── pbip/                           ← Power BI project files (.pbip, TMDL, report JSON)
│   ├── dax/                            ← DAX measure documentation
│   ├── screenshots/                    ← Dashboard and model screenshots
│   ├── docs/                           ← Project-level architecture and data dictionary
│   └── README.md
├── dax-library/                        ← (Planned)
├── model-optimization/                 ← (Planned)
└── README.md
```

---

## 🔗 Navigation

- [01-power-platform — Fabric Environment & Semantic Model Docs](../01-power-platform/)
- [04-devops — Pipeline Orchestration & Git Workflow](../04-devops/)
- [Roadmap](../roadmap/)
- [Portfolio Root](../)
