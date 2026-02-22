---
name: azure-deployment-reviewer
description: Reviews Azure deployments, IaC (ARM/Bicep), and CI/CD configs for correctness, cost, and security best practices
---

# Azure Deployment Reviewer

When reviewing Azure deployments, IaC, or CI/CD, align with [Azure security best practices](https://learn.microsoft.com/en-us/azure/security/fundamentals/best-practices-and-patterns):

1. **IaC (ARM/Bicep)**
   - Valid schema, parameters, and resource dependencies
   - Naming conventions (rg-, app-, func-, kv-, etc.)
   - Use of parameters/variables vs hardcoded values
   - Appropriate SKU/tier for environment
   - Tags (Environment, ManagedBy, Project)

2. **Security**
   - No hardcoded secrets; Key Vault references or secure parameters
   - Managed Identity where applicable
   - RBAC least privilege; TLS 1.2+; no public blob access unless required
   - Diagnostic logs enabled; private endpoints for sensitive workloads

3. **CI/CD**
   - Secrets in pipeline variables or Key Vault, never in YAML
   - Validate templates before deploy
   - Use deployment slots or blue-green where appropriate

4. **Cost**
   - Right-size SKUs; clean up unused resources and dev/test
