# Azure Administration & Engineering

## 📌 Overview

This section documents Azure administration and infrastructure projects aligned to the **AZ-104** certification track and supporting the broader cloud engineering roadmap. Projects will cover the core Azure administrator domains: identity and access management, virtual networking, storage, compute, monitoring, and governance — with Infrastructure as Code (Bicep and/or Terraform) applied throughout.

Content in this section is planned for development following PL-300 and PL-200 completion. Projects will be built as hands-on labs with full documentation, architecture diagrams, and lessons learned, mirroring the engineering standards applied in the existing Power Platform and Fabric sections.

---

## 📂 Planned Projects

### 1. Identity & Access Management Lab
Entra ID configuration, RBAC role assignments, managed identities, and conditional access policies. Demonstrates secure identity architecture relevant to both standalone Azure environments and Power Platform / Fabric integrations.

### 2. Virtual Network Design Lab
VNET creation, subnet segmentation, peering, Network Security Groups, and DNS configuration. Includes a network diagram and Bicep/Terraform deployment templates.

### 3. Azure Governance & Cost Management
Management group hierarchy, policy assignments, resource tagging strategy, cost analysis, and budget alerts. Aligned to enterprise governance patterns applicable to Fabric capacity management and Power Platform environment governance.

### 4. Storage & Compute Lab
Storage account configuration (blob, file, table, queue), lifecycle management, Azure VM deployment, and availability sets. Includes IaC templates and configuration documentation.

### 5. Monitoring & Diagnostics
Azure Monitor, Log Analytics workspace configuration, diagnostic settings, alert rules, and workbook dashboards. Directly applicable to monitoring Fabric pipeline runs and Power Platform connector activity.

---

## 🧩 Architecture Approach

Each lab will follow a consistent pattern:

```
IaC Template (Bicep / Terraform)
        │  provision
        ▼
Azure Resource Group
  ├── Core resources (per lab scope)
  ├── Diagnostic settings → Log Analytics
  └── RBAC assignments
        │
        ▼
Architecture diagram + deployment notes + lessons learned
```

All lab environments will be deployed via IaC to ensure reproducibility and version control. Manual portal steps will be documented only where IaC equivalents are not available.

---

## 📂 Planned Folder Structure

```
02-azure-admin/
├── 01-identity-rbac/
│   ├── docs/
│   ├── iac/
│   ├── screenshots/
│   └── README.md
├── 02-virtual-networking/
│   ├── docs/
│   ├── iac/
│   ├── screenshots/
│   └── README.md
├── 03-governance-cost/
│   ├── docs/
│   ├── screenshots/
│   └── README.md
├── 04-storage-compute/
│   ├── docs/
│   ├── iac/
│   ├── screenshots/
│   └── README.md
├── 05-monitoring/
│   ├── docs/
│   ├── screenshots/
│   └── README.md
└── README.md
```

---

## 🎓 Related Certifications

| Certification | Status | Relevance |
|---|---|---|
| AZ-104 — Azure Administrator Associate | 📋 Planned 2027 | Identity, networking, storage, compute, monitoring |
| AZ-400 — Azure DevOps Engineer Expert | 📋 Planned 2027 | IaC pipelines, deployment automation |
| DP-300 — Azure Database Administrator Associate | 📋 Planned Q4 2026 | Azure SQL, managed instances, backup and recovery |

---

## 🔗 Navigation

- [Portfolio Root](../)
- [01-power-platform — Fabric & Power Platform](../01-power-platform/)
- [04-devops — Pipeline & IaC (planned)](../04-devops/)
- [Roadmap](../roadmap/)
