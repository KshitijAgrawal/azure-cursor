---
name: azure-functions
description: Create, configure, and deploy Azure Functions. Use when working with Azure Functions, function apps, serverless Azure, triggers (HTTP, timer, queue, blob), or when the user asks about Azure Functions, Durable Functions, or function app configuration.
---

# Azure Functions Skill

## Quick Start

Azure Functions run event-driven, serverless code. Common triggers: HTTP, Timer, Blob, Queue, Event Hub, Service Bus.

## Project Structure (v4)

```
func-app/
├── host.json
├── local.settings.json
├── function.json (or using annotations)
└── <language-specific>/
```

## host.json

```json
{
  "version": "2.0",
  "logging": {
    "logLevel": {
      "default": "Information",
      "Host.Results": "Error"
    }
  },
  "extensionBundle": {
    "id": "Microsoft.Azure.Functions.ExtensionBundle",
    "version": "[4.*, 5.0.0)"
  },
  "functionTimeout": "00:05:00"
}
```

## Function App Naming

- Use lowercase, alphanumeric, hyphens: `func-myapp-prod-eastus2`
- Must be globally unique (add suffix if needed)

## Key Configuration

| Setting | Purpose |
|---------|---------|
| `FUNCTIONS_WORKER_RUNTIME` | node, python, dotnet, java |
| `AzureWebJobsStorage` | Required; Blob/Queue connection for internal ops |
| `WEBSITE_RUN_FROM_PACKAGE` | 1 = run from deployment package |
| `FUNCTIONS_EXTENSION_VERSION` | ~4 for v4 runtime |

## Triggers by Use Case

| Trigger | Use Case |
|---------|----------|
| HTTP | APIs, webhooks |
| Timer | Scheduled jobs (cron) |
| Blob | Process file uploads |
| Queue | Async processing |
| Event Hub | Event streaming |
| Service Bus | Message queues |
| Cosmos DB | Change feed |

## Bicep: Function App

```bicep
param functionAppName string
param storageAccountName string
param appServicePlanId string
param runtime string = 'node'

resource functionApp 'Microsoft.Web/sites@2022-09-01' = {
  name: functionAppName
  kind: 'functionapp'
  location: resourceGroup().location
  properties: {
    serverFarmId: appServicePlanId
    siteConfig: {
      appSettings: [
        { name: 'AzureWebJobsStorage', value: 'DefaultEndpointsProtocol=https;AccountName=${storageAccountName};...' }
        { name: 'FUNCTIONS_WORKER_RUNTIME', value: runtime }
        { name: 'WEBSITE_NODE_DEFAULT_VERSION', value: '~18' }
        { name: 'FUNCTIONS_EXTENSION_VERSION', value: '~4' }
      ]
    }
  }
}
```

## Best Practices

1. **Use Managed Identity** instead of connection strings when possible
2. **Store secrets in Key Vault** and reference via App Settings
3. **Set appropriate timeouts** (max 10 min for Consumption, higher for Premium)
4. **Use Durable Functions** for orchestrations and long-running workflows
5. **Configure scaling** (Consumption auto-scales; Premium/ASE for VNet, longer timeouts)
