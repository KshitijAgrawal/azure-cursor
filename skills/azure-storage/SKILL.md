---
name: azure-storage
description: Create and configure Azure Storage accounts. Use when working with Storage accounts, Blob, Queue, Table, File storage, connection strings, or when the user asks about Azure Storage, blob containers, storage Bicep/ARM, or storing files/data in Azure.
---

# Azure Storage Skill

## Quick Start

Azure Storage provides Blob (objects), Queue (messaging), Table (key-value), and File (SMB) storage. A Storage Account is required for Blob, Queue, Table; File shares live inside it.

## Naming

- **Storage Account**: 3–24 chars, lowercase letters and numbers only, globally unique
- Use `uniqueString(resourceGroup().id)` for uniqueness: `st${uniqueString(resourceGroup().id)}`

## SKUs

| SKU | Use Case |
|-----|----------|
| Standard_LRS | Locally redundant, cost-effective |
| Standard_GRS | Geo-redundant, backup |
| Standard_ZRS | Zone-redundant, high availability |
| Premium_LRS | Premium block blobs, low latency |

## Bicep: Storage Account

```bicep
param storageAccountName string = 'st${uniqueString(resourceGroup().id)}'
param location string = resourceGroup().location

resource storageAccount 'Microsoft.Storage/storageAccounts@2023-01-01' = {
  name: storageAccountName
  location: location
  sku: { name: 'Standard_LRS' }
  kind: 'StorageV2'
  properties: {
    supportsHttpsTrafficOnly: true
    minimumTlsVersion: 'TLS1_2'
    allowBlobPublicAccess: false
  }
}
```

## Bicep: Storage Account + Blob Container

```bicep
resource blobService 'Microsoft.Storage/storageAccounts/blobServices@2023-01-01' = {
  parent: storageAccount
  name: 'default'
}

resource container 'Microsoft.Storage/storageAccounts/blobServices/containers@2023-01-01' = {
  parent: blobService
  name: 'mycontainer'
  properties: {
    publicAccess: 'None'
  }
}
```

## Bicep: Queue

```bicep
resource queueService 'Microsoft.Storage/storageAccounts/queueServices@2023-01-01' = {
  parent: storageAccount
  name: 'default'
}

resource queue 'Microsoft.Storage/storageAccounts/queueServices/queues@2023-01-01' = {
  parent: queueService
  name: 'myqueue'
}
```

## Connection String

Format: `DefaultEndpointsProtocol=https;AccountName=<name>;AccountKey=<key>;EndpointSuffix=core.windows.net`

Store in Key Vault or app settings; never commit to source control.

## Managed Identity

Use managed identity instead of connection strings when the app runs in Azure (App Service, Functions, VM). Grant the identity **Storage Blob Data Contributor**, **Storage Queue Data Contributor**, etc. via RBAC.

## Access Keys

```bash
az storage account keys list --account-name <name> --resource-group <rg> -o table
```

Rotate keys periodically; use Key Vault references for app config.

## Common Commands

```bash
# Create storage account
az storage account create --name <name> --resource-group <rg> --sku Standard_LRS

# Create blob container
az storage container create --account-name <name> --name mycontainer

# Create queue
az storage queue create --account-name <name> --name myqueue
```

## Best Practices

1. **Disable public blob access** (`allowBlobPublicAccess: false`) unless needed
2. **Use managed identity** for Azure-hosted apps
3. **Store connection strings in Key Vault** when keys are required
4. **Enable TLS 1.2** minimum
5. **Use appropriate redundancy** (LRS for dev, GRS/ZRS for prod)
