---
name: validate-iac
description: Validate Bicep or ARM templates before deployment
---

# Validate Azure IaC

Ensure user is logged in (`az login`) and correct subscription is active. If restricted mode is readOnly, validation (az bicep build, az deployment group validate) is allowed; deployment is not.

When validating Bicep or ARM templates:

**Bicep:**
```bash
az bicep build --file main.bicep
```

**ARM (resource group):**
```bash
az deployment group validate \
  --resource-group <rg-name> \
  --template-file azuredeploy.json \
  --parameters @parameters.json
```

**ARM (subscription):**
```bash
az deployment sub validate \
  --location eastus \
  --template-file main.json
```

Report any validation errors (syntax, missing parameters, invalid resource types) and suggest fixes.
