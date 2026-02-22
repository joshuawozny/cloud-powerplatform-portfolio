# ⚙️ DevOps Pipeline Naming Standards

This document defines naming conventions for Azure DevOps and GitHub Actions pipelines.

---

## 🔤 General Rules
- Use **snake_case** for pipeline files.
- Use **lowercase** for folder names.
- Use descriptive action + target naming.

---

## 🧩 Pipeline File Naming
```
deploy_powerplatform_solution.yml
build_and_test_functionapp.yml
iac_provisioning_pipeline.yml
```

---

## 📂 Folder Structure
```
/pipelines
  deploy_powerplatform_solution.yml
  build_and_test_functionapp.yml
/iac
  main.bicep
  variables.json
```

---

## 🧪 Pipeline Naming in Azure DevOps UI
Use PascalCase:
```
Deploy-PowerPlatform-Solution
BuildAndTest-FunctionApp
Provision-IaC
```

---

## 🎯 Summary
This standard ensures:
- Predictable CI/CD structure  
- Clean automation workflows  
- Easy navigation for reviewers  
