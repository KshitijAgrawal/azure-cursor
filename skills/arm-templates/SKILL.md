---
name: arm-templates
description: Author, validate, and deploy Azure ARM (JSON) templates. Use when creating or editing ARM templates, azuredeploy.json, ARM JSON files, Azure Resource Manager templates, or when the user asks about ARM template structure, parameters, or best practices.
---

# ARM Templates Skill

## Quick Start

When working with ARM templates (JSON):

1. Ensure valid schema and structure
2. Use parameters for environment-specific values
3. Use variables for repeated expressions
4. Follow Azure naming conventions

## Template Structure

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {},
  "variables": {},
  "resources": [],
  "outputs": {}
}
```

- **$schema**: Use `deploymentTemplate.json#` for resource group, `deploymentTemplate.json` for subscription. Always use HTTPS schema URL.
- **contentVersion**: Semantic version (e.g., `1.0.0.0`)

## Parameters

- Use camelCase for parameter names
- Add `metadata.description` to every parameter
- Use `securestring` or `secureObject` for secrets (passwords, connection strings, keys)
- Prefer `allowedValues` for SKU, tier, or limited options
- Set sensible `defaultValue` when safe

```json
{
  "adminPassword": {
    "type": "securestring",
    "metadata": {
      "description": "Admin password for the VM"
    }
  },
  "sku": {
    "type": "string",
    "defaultValue": "Standard_B2s",
    "allowedValues": ["Standard_B2s", "Standard_D2s_v3"],
    "metadata": { "description": "VM SKU" }
  }
}
```

## Variables

- Use variables for complex expressions or values used more than once
- Keep template expressions under 24,576 characters
- Use `variables()` and `parameters()` in expressions

## Resources

- Order resources by dependency (consumers after providers)
- Use `dependsOn` for implicit dependencies
- Use `existing` deployments to reference existing resources when appropriate

## Limits

- Template size: 4 MB max
- Single resource definition: 1 MB max
- Max 256 parameters, 256 variables, 800 resources, 64 outputs

## Best Practices

1. **Modularize**: Split large templates into linked templates or use Bicep modules
2. **Location**: Prefer `[resourceGroup().location]` for resource locations
3. **Naming**: Use `uniqueString()` for globally unique names when needed
4. **Outputs**: Output resource IDs, endpoints, and connection strings for downstream use

## Common Resource Types

| Resource | Type |
|----------|------|
| Storage Account | `Microsoft.Storage/storageAccounts` |
| App Service Plan | `Microsoft.Web/serverfarms` |
| Web App | `Microsoft.Web/sites` |
| Function App | `Microsoft.Web/sites` (kind: functionapp) |
| Key Vault | `Microsoft.KeyVault/vaults` |
| Cosmos DB | `Microsoft.DocumentDB/databaseAccounts` |

## Validation

Before deployment:
- Run `az deployment group validate` for resource group deployments
- Use ARM template tools extension in VS Code for intellisense
