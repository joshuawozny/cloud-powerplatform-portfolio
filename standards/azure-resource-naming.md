# ☁️ Azure Resource Naming Standards (CAF-Aligned)

This naming convention follows the Microsoft Cloud Adoption Framework (CAF) and ensures consistency, clarity, and scalability across Azure resources used in this portfolio.

## 🔤 General Rules
- Use **lowercase** only.
- Use **short, CAF-aligned resource type abbreviations**.
- Use **hyphens** to separate segments.
- Avoid environment names unless necessary (dev/test/prod).
- Keep names under Azure’s character limits.

## 🧩 Naming Pattern

```
<project>-<resource-type>-<instance>
```
**Examples:**
- `nycdata-func-api`
- `ppsolution-sql-db1`
- `capstone-vnet-hub`

## 📘 Common Azure Resource Abbreviations (CAF Standard)
| Resource Type | Abbreviation | Example |
|---------------|--------------|---------|
| Resource Group | rg | `capstone-rg` |
| Virtual Network | vnet | `capstone-vnet-hub` |
| Subnet | snet | `capstone-snet-app` |
| Storage Account | st | `capstone-st01` |
| Function App | func | `nycdata-func-api` |
| App Service | app | `capstone-app-web` |
| SQL Database | sql | `ppsolution-sql-db1` |
| Key Vault | kv | `capstone-kv` |
| Log Analytics | log | `capstone-log` |

---

## 🧱 Environment Suffixes (Optional)
Use only when needed:
- `-dev`
- `-test`
- `-prod`

Example:
```
capstone-func-api-dev
```
## 🎯 Summary
CAF-aligned naming ensures:
- Predictable resource discovery  
- Easier automation and IaC  
- Cleaner Azure portal organization  
---
