---
name: bicep
description: Author and deploy Azure Bicep Infrastructure as Code. Use when creating or editing .bicep files, Bicep modules, Azure IaC with Bicep, or when the user prefers Bicep over ARM JSON for Azure deployments.
---

# Bicep Skill

## Quick Start

Bicep is a DSL that transpiles to ARM. Use Bicep for:
- Cleaner, more readable IaC
- Native Azure support (no state file)
- Loops, conditionals, and modules without JSON verbosity

## Basic Structure

```bicep
@description('Primary location for resources')
param location string = resourceGroup().location

@description('Environment name')
param environment string = 'dev'

param tags object = {
  Environment: environment
  ManagedBy: 'Bicep'
  'created-by-cursor-azure': 'true'
}

resource storageAccount 'Microsoft.Storage/storageAccounts@2023-01-01' = {
  name: uniqueString(resourceGroup().id)
  location: location
  tags: tags
  sku: {
    name: 'Standard_LRS'
  }
  kind: 'StorageV2'
}

output storageId string = storageAccount.id
```

## Parameters

- Use `@description()` decorator for documentation
- Use `@secure()` for sensitive parameters
- Use `@allowed()` for constrained values

```bicep
@description('Admin password')
@secure()
param adminPassword string

@allowed(['dev', 'staging', 'prod'])
param environment string = 'dev'
```

## Variables

```bicep
var storageAccountName = 'st${uniqueString(resourceGroup().id)}'
var appServicePlanName = 'asp-${resourceGroup().name}-${environment}'
```

## Conditional Resources

```bicep
resource keyVault 'Microsoft.KeyVault/vaults@2023-07-01' = if (environment != 'dev') {
  name: 'kv-${uniqueString(resourceGroup().id)}'
  location: location
  properties: {
    sku: { name: 'standard', family: 'A' }
    tenantId: subscription().tenantId
    enabledForDeployment: true
  }
}
```

## Loops

```bicep
param environments array = ['dev', 'staging', 'prod']

resource storageAccounts 'Microsoft.Storage/storageAccounts@2023-01-01' = [for env in environments: {
  name: 'st${uniqueString(resourceGroup().id)}${env}'
  location: location
  sku: { name: 'Standard_LRS' }
  kind: 'StorageV2'
}]
```

## Modules

```bicep
// main.bicep
module storageModule 'modules/storage.bicep' = {
  name: 'storageDeployment'
  params: {
    location: location
    environment: environment
  }
}

output storageId string = storageModule.outputs.storageId
```

```bicep
// modules/storage.bicep
param location string
param environment string

resource sa 'Microsoft.Storage/storageAccounts@2023-01-01' = {
  name: 'st${uniqueString(resourceGroup().id)}'
  location: location
  sku: { name: 'Standard_LRS' }
  kind: 'StorageV2'
}

output storageId string = sa.id
```

## Deployment Commands

```bash
# Validate
az bicep build --file main.bicep

# Deploy (resource group)
az deployment group create --resource-group myRG --template-file main.bicep --parameters environment=prod

# Deploy (subscription)
az deployment sub create --location eastus --template-file main.bicep
```

## Best Practices

1. **Prefer Bicep over ARM JSON** for new Azure-only IaC
2. **Use modules** for reusable components (storage, networking, app services)
3. **Parameterize** environment, SKU, and capacity
4. **Add tags** to all resources (Environment, ManagedBy, Project, created-by-cursor-azure: 'true')
5. **Use existing** keyword to reference existing resources instead of creating duplicates
