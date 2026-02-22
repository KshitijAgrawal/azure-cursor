---
name: generate-bicep
description: Generate Bicep or ARM template for an Azure resource or service
---

# Generate Azure IaC

When the user asks to generate Bicep or ARM for an Azure resource:

1. **Clarify resource**: Storage Account, App Service, Function App, Key Vault, Cosmos DB, etc.
2. **Use Bicep by default** for new IaC; use ARM JSON only if explicitly requested or for legacy.
3. **Include**:
   - Parameters for name, location, SKU, environment
   - Outputs for resource IDs or connection endpoints
   - Tags (Environment, ManagedBy, created-by-cursor-azure: 'true')
4. **Follow** resource provider API versions from Azure docs (e.g., `@2022-09-01` for Web).
5. **Reference** the ARM/Bicep skills in this plugin for structure and best practices.
