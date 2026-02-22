# 🧩 Power Platform Naming Standards

This document defines naming conventions for Power Apps, Dataverse tables, columns, solutions, flows, and plugins.

---

## 📦 Solution Naming
Use PascalCase with domain + purpose.
```
ProjectName_Core ProjectName_Integrations ProjectName_Admin
```

**Examples:**
- `NYCData_Core`
- `Capstone_Integrations`

---

## 🗄️ Dataverse Table Naming
Use **singular nouns** in PascalCase.
```
Request
Agency
ComplaintType
Location
```

### Display Name vs Schema Name
- **Display Name:** `Request`
- **Schema Name:** `crd_Request`

Prefix:  
Use a unique publisher prefix such as `crd` (Cloud Research & Development).

---

## 🔢 Column Naming
Use PascalCase for display names, camelCase for schema names.

**Display Name:**  
`Resolution Hours`

**Schema Name:**  
`crd_resolutionHours`

---

## 🔄 Power Automate Flow Naming
Use snake_case with action + target.
```
create_request_flow
sync_agency_data_flow
notify_overdue_requests_flo
```

---

## 🧩 Plugin Naming
Use PascalCase with purpose.
```
RequestValidationPlugin.cs
AgencyAssignmentPlugin.cs
```

---

## 🎯 Summary
These standards ensure:
- Clean Dataverse schema  
- Predictable automation naming  
- Professional solution structure  
