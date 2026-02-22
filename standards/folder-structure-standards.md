# 📁 Folder Structure Standards

This document defines the folder structure conventions used across this portfolio.  
The goal is to maintain clarity, scalability, and consistency as projects grow across Power Platform, Azure, DevOps, and Data Analytics domains.

---

## 🔤 General Principles
- Use **lowercase-with-hyphens** for all folder names.
- Keep folder names **short, descriptive, and technology‑aligned**.
- Avoid dates, personal identifiers, or environment-specific names.
- Each project folder must contain a **README.md**.
- Use consistent subfolder patterns across all domains.

---

## 🧱 Top-Level Portfolio Structure
---
/01-power-platform 
/02-azure-admin 
/03-data-analytics 
/04-devops 
/05-agile-pmp 
/06-capstone /standards
---
## 📂 Standard Project Folder Structure
Each project folder should follow this pattern:
---
/project-name 
README.md 
  /src 
  /docs 
  /screenshots 
  /pbip (if Power BI) 
  /dax (if Power BI) 
  /iac (if Azure/DevOps) 
  /pipelines (if DevOps)
---

## 🧩 Domain-Specific Folder Rules

### **Power Platform**
---
/01-power-platform/project-name 
  /src 
  /docs 
  /screenshots
---
### **Azure**
---
/02-azure-admin/project-name 
  /iac 
  /docs 
  /screenshots
---
### **Data Analytics**
---
/03-data-analytics/project-name 
  /pbip 
  /dax 
  /screenshots 
  /docs
---
### **DevOps**
---
/04-devops/project-name 
  /pipelines 
  /iac 
  /docs
---
### **Agile/PMP**
---
/05-agile-pmp/project-name 
  /docs 
  /screenshots

---

## 🎯 Summary
This structure ensures:
- Predictable navigation  
- Professional documentation  
- Clean Git history  
- Easy onboarding for reviewers and hiring managers  
