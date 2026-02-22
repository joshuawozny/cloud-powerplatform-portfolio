# 📁 File Naming Standards

To maintain clarity and consistency across this portfolio, all files follow a structured naming convention. These standards ensure that projects remain easy to navigate, understand, and maintain — especially as the portfolio expands across Power Platform, Azure, DevOps, and Data Analytics domains.

---

## 🔤 General Rules
- Use **PascalCase** or **snake_case** (choose based on file type below)
- Avoid spaces in file names
- Avoid personal identifiers (e.g., your name)
- Avoid dates unless versioning is required
- Keep names descriptive but concise
- Use lowercase for folders, uppercase for acronyms (e.g., `API`, `DAX`, `PBIP`)

---

## 📊 Power BI Files
**Format:**  
`ProjectName.pbip`  
or  
`ProjectName.pbix` (if using PBIX)

**Examples:**  
- `NYC_Open_Data_Dashboard.pbip`  
- `Transportation_Insights.pbip`

**Rationale:**  
The project folder already provides context, so the file name stays clean and focused.

---

## 🧩 Power Platform Files
### **Power Apps**
Use the app name in PascalCase:  
`AppName.msapp`

### **Power Automate**
Use snake_case with action focus:  
`create_case_flow.json`  
`sync_dataverse_records_flow.json`

### **Dataverse Plugins**
Use PascalCase with plugin purpose:  
`CaseAssignmentPlugin.cs`  
`ValidationPlugin.cs`

### **PCF Components**
Use PascalCase for component name:  
`AddressLookupControl`  
`StatusBadgeComponent`

---

## ☁️ Azure Files
### **Bicep / Terraform**
Use resource‑focused snake_case:  
`storage_account.bicep`  
`vnet_peering.tf`

### **Azure Functions**
Use PascalCase for function folders:  
`ProcessRequestsFunction/`  
`SyncRecordsFunction/`

---

## ⚙️ DevOps Pipelines
Use snake_case with pipeline purpose:  
`deploy_powerplatform_solution.yml`  
`iac_provisioning_pipeline.yml`

---

## 📈 Data Analytics Files
### **DAX Measures**
Use descriptive snake_case:  
`total_requests.dax`  
`avg_resolution_time.dax`

### **Power Query Scripts**
`transform_requests.m`  
`clean_locations.m`

### **Screenshots**
Use lowercase snake_case with sequence numbers:  
`screenshot_overview_01.png`  
`screenshot_trendline_02.png`

---

## 📄 Documentation Files
### **Project READMEs**
Always named:  
`README.md`

### **Supporting Docs**
Use snake_case:  
`architecture_diagram.png`  
`data_model.png`  
`lessons_learned.md`

---

## 🧱 Folder Naming
Use lowercase with hyphens:  
- `nyc-open-data-dashboard`  
- `dataverse-plugin`  
- `vnet-peering-lab`  
- `ci-cd-pipeline`

This keeps URLs clean and predictable.

---

## 🎯 Versioning (Optional)
If you need explicit versioning:  
`ProjectName_v1.pbip`  
`ProjectName_v2.pbip`

Use only when maintaining multiple historical versions.

---

## ✔️ Summary
This naming system ensures:
- Consistency across all domains  
- Easy navigation for recruiters and hiring managers  
- Professional engineering discipline  
- Clean Git history and predictable file paths  
