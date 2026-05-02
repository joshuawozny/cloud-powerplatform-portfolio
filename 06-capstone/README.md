# Capstone — Integrated Enterprise Solution

## 📌 Overview

This section will contain the portfolio's flagship project — a full end-to-end enterprise solution integrating Power Platform, Microsoft Fabric, Azure, DevOps, and formal project management documentation. The capstone is planned for 2027 as the culmination of the certification roadmap and the MS in Information Technology program.

Where the individual domain sections (`01` through `05`) demonstrate depth in specific technology areas, the capstone demonstrates breadth — the ability to architect, build, deploy, govern, and document a production-grade solution across the full cloud and platform engineering stack.

---

## 🧩 Planned Architecture

The capstone will build on the foundation established by the NYC311 Fabric Data Platform and extend it — or define a new enterprise scenario — to incorporate components not yet present in the portfolio. The exact scenario is to be determined, but the architectural scope will cover:

```
Power Platform
  ├── Canvas or Model-Driven App (user-facing)
  ├── Dataverse (data layer with custom tables and relationships)
  ├── C# Plugin (server-side business logic)
  ├── PCF Component (custom UI control)
  └── Power Automate (approval and integration flows)

Microsoft Fabric / Azure Data
  ├── Lakehouse (medallion architecture)
  ├── Pipelines & Notebooks (orchestrated data engineering)
  └── Power BI Semantic Model (mixed-mode, aggregations)

Azure Infrastructure
  ├── Azure Functions (serverless API / integration layer)
  ├── Azure SQL or Cosmos DB (transactional data store)
  ├── Entra ID (identity, RBAC, managed identity)
  └── Key Vault, Monitor, Log Analytics (security & observability)

DevOps
  ├── Azure DevOps YAML pipelines (CI/CD for Power Platform ALM)
  ├── IaC (Bicep or Terraform for Azure resource provisioning)
  └── Git branching strategy with environment-gated deployment

Project Management
  ├── Project charter and scope statement
  ├── WBS and schedule baseline
  ├── Risk register and change log
  └── Sprint artifacts and retrospectives
```

---

## 🎯 Purpose & Goals

**Technical:** Demonstrate that all certification-track skills can be integrated into a single coherent solution rather than existing in isolated labs. A senior reviewer should be able to trace a user interaction from the front-end app through business logic, data persistence, integration, analytics, and deployment — with governance and documentation at every layer.

**Professional:** Provide a case study anchor for technical interviews and consulting engagements. The capstone scenario will be chosen to reflect the type of work done at IncWorx — enterprise clients, multi-system integration, governance requirements — rather than a purely academic exercise.

**Certification alignment:** The capstone is explicitly designed so that completing it constitutes applied evidence for PL-400, AZ-104, AZ-400, and PMP competencies simultaneously.

---

## 📅 Timeline

| Milestone | Target |
|---|---|
| Domain sections (`01`–`05`) substantially complete | Mid-2026 |
| Capstone scenario defined and scoped | Q4 2026 |
| Architecture and project documentation | Q1 2027 |
| Build and deployment | Q1–Q2 2027 |
| Final documentation, case study, and lessons learned | Q2 2027 |

---

## 📂 Planned Folder Structure

```
06-capstone/
├── docs/
│   ├── project-charter.md
│   ├── architecture.md          ← Current + future state diagrams
│   ├── data-flow.md
│   ├── component-map.md
│   ├── risk-register.md
│   └── lessons-learned.md
├── power-platform/
│   ├── canvas-app/
│   ├── dataverse-schema/
│   ├── plugin/
│   └── pcf-component/
├── fabric/
│   ├── notebooks/
│   ├── pipeline/
│   └── semantic-model/
├── azure/
│   ├── functions/
│   └── iac/
├── devops/
│   ├── pipelines/
│   └── environments/
├── agile-docs/
│   ├── sprint-plans/
│   └── retrospectives/
└── README.md
```

---

## 🎓 Certifications Demonstrated

| Certification | Relevance |
|---|---|
| PL-400 — Power Platform Developer | C# Plugin, PCF Component, ALM pipeline |
| AZ-104 — Azure Administrator | Identity, networking, storage, compute |
| AZ-400 — Azure DevOps Engineer | CI/CD pipelines, IaC, release management |
| DP-300 — Azure Database Administrator | Azure SQL / Cosmos DB design and management |
| PMP — Project Management Professional | Charter, WBS, risk, schedule, change control |

---

## 🔗 Navigation

- [Portfolio Root](../)
- [01-power-platform — Fabric & Power Platform](../01-power-platform/)
- [03-data-analytics — NYC311 Data Platform](../03-data-analytics/)
- [04-devops — Pipeline & Source Control](../04-devops/)
- [05-agile-pmp — Project Management Artifacts](../05-agile-pmp/)
- [Roadmap](../roadmap/)
