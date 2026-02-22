---
name: azure-app-service
description: Configure and deploy Azure App Service (Web Apps, APIs). Use when deploying web apps, configuring App Service, Azure Web App, App Service Plan, deployment slots, custom containers, scaling, or when the user asks about hosting .NET, Node.js, Python, or Java apps on Azure.
---

# Azure App Service Skill

## Quick Start

App Service hosts web apps, APIs, and mobile backends. Supports .NET, Node.js, Python, Java, and custom containers.

## Naming Conventions

- **Web App**: `app-<name>-<env>` (e.g., `app-myapi-prod`)
- **App Service Plan**: `asp-<name>-<env>` (e.g., `asp-myapi-prod`)

## SKU Tiers

| Tier | Use Case | Scaling |
|------|----------|---------|
| Free / Shared | Dev, testing | No scale-out |
| Basic | Dev/test, low traffic | Manual scale |
| Standard | Production | Auto-scale, slots |
| Premium | Production, VNet, slots | Auto-scale, more slots |
| Isolated | High compliance | Dedicated ASE |

## Key App Settings

| Setting | Purpose |
|---------|---------|
| `WEBSITE_NODE_DEFAULT_VERSION` | Node version (e.g., `~18`) |
| `WEBSITE_RUN_FROM_PACKAGE` | Run from zip (`1`) – faster, immutable |
| `SCM_DO_BUILD_DURING_DEPLOYMENT` | Build during deploy (true/false) |
| `ApplicationInsightsInstrumentationKey` | APM with Application Insights |
| `WEBSITE_ENABLE_SYNC_UPDATE_SITE` | Sync deployment (true/false) |

## Deployment Options

1. **ZIP Deploy**: `az webapp deployment source config-zip`
2. **Git/GitHub**: Connect repo, Kudu auto-deploys
3. **Container**: `linuxFxVersion: 'DOCKER|<image>'` or `windowsFxVersion`
4. **FTP / Local Git**: Alternative methods

## Bicep: Web App + App Service Plan (Node.js)

```bicep
param webAppName string
param nodeVersion string = '~18'

resource plan 'Microsoft.Web/serverfarms@2022-09-01' = {
  name: '${webAppName}-plan'
  location: resourceGroup().location
  sku: { name: 'B1', tier: 'Basic' }
  kind: 'linux'
  properties: { reserved: true }
}

resource webApp 'Microsoft.Web/sites@2022-09-01' = {
  name: webAppName
  location: resourceGroup().location
  kind: 'app,linux'
  properties: {
    serverFarmId: plan.id
    siteConfig: {
      linuxFxVersion: 'NODE|${nodeVersion}'
      appSettings: [
        { name: 'WEBSITE_NODE_DEFAULT_VERSION', value: nodeVersion }
      ]
    }
  }
}
```

## Bicep: Web App with Custom Container

```bicep
resource webApp 'Microsoft.Web/sites@2022-09-01' = {
  name: webAppName
  location: resourceGroup().location
  kind: 'app,linux'
  identity: { type: 'SystemAssigned' }
  properties: {
    serverFarmId: plan.id
    siteConfig: {
      linuxFxVersion: 'DOCKER|mcr.microsoft.com/azuredocs/containerapps-helloworld:latest'
    }
  }
}
```

## Key Vault References (App Settings)

Store secrets in Key Vault and reference from App Service:

```bicep
appSettings: [
  {
    name: 'MySecret'
    value: '@Microsoft.KeyVault(SecretUri=https://kv-myapp.vault.azure.net/secrets/MySecret/)'
  }
]
```

Enable Key Vault access: assign `Key Vault Secrets User` (or `Get` permission) to the app’s managed identity.

## Deployment Slots

- Use for staging → production swaps without downtime
- Slot-specific app settings: configure per slot
- Swap: `az webapp deployment slot swap --name <app> --resource-group <rg> --slot staging --target-slot production`

```bicep
resource stagingSlot 'Microsoft.Web/sites/slots@2022-09-01' = {
  parent: webApp
  name: 'staging'
  location: resourceGroup().location
  properties: {
    serverFarmId: plan.id
  }
}
```

## Auto-Scaling (Standard+)

Configure in Bicep via `Microsoft.Insights/autoscaleSettings` or Portal. Scale out by CPU, HTTP queue length, or custom metrics.

## Health Checks

Set `healthCheckPath` in `siteConfig` (e.g., `/health`) for load balancer health probes.

## Best Practices

1. **Use deployment slots** for staging/prod swaps
2. **Store secrets in Key Vault** and use Key Vault references
3. **Enable Application Insights** for monitoring
4. **Use managed identity** for Azure resource access
5. **Set `WEBSITE_RUN_FROM_PACKAGE`** for faster, immutable deploys
6. **Enable HTTPS only** and use managed certificates
