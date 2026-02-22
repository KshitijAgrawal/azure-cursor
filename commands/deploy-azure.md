---
name: deploy-azure
description: Deploy application or infrastructure to Azure
---

# Deploy to Azure

When the user asks to deploy to Azure, follow these steps:

1. **Check restricted mode**: If `azure-cursor-community.config.json` or `.cursor/azure-cursor-community.json` exists with `restrictedMode: true`, enforce scope (readOnly or resourceGroup). See azure-restricted-mode rule. Do not deploy if scope is readOnly.
2. **Ensure Azure login**: If not logged in, run `az login` (or `az login --use-device-code` for headless). Verify with `az account show`. Reference the azure-authentication skill for service principal, subscription selection, or troubleshooting.
3. **Identify target**: Web App, Function App, Static Web App, container, or IaC (Bicep/ARM).
4. **Check for existing config**: Look for `azure-pipelines.yml`, GitHub Actions workflows, `main.bicep`, `azuredeploy.json`, or deployment scripts.
5. **Proceed with appropriate method**:
   - **Web/Function App**: Use `az webapp deployment source config-zip` or ZIP deploy; ensure `host.json`, `function.json`, and app code are included.
   - **Bicep/ARM**: Run `az bicep build` (Bicep) or `az deployment group validate` (ARM) first, then `az deployment group create`.
   - **Static Web App**: Use `az staticwebapp create` and `swa deploy` or GitHub Action.
6. **Apply default tag**: Add `created-by-cursor-azure: "true"` to all resources created (resource groups, storage, web apps, etc.) that support tags. For `az group create`, use `--tags created-by-cursor-azure=true`. For Bicep/ARM, include in the tags object.
7. **Verify**: Confirm deployment succeeded and endpoints are reachable.
