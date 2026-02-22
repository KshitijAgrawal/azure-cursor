---
name: azure-static-web-apps
description: Create and deploy Azure Static Web Apps. Use when working with Static Web Apps, SPAs, JAMstack, React/Vue/Angular/Svelte on Azure, or when the user asks about hosting static sites, SWA, or frontend-only apps on Azure.
---

# Azure Static Web Apps Skill

## Quick Start

Azure Static Web Apps hosts static sites and SPAs with optional serverless API (Azure Functions). Supports React, Vue, Angular, Svelte, Next.js static export, and other static generators.

## Naming

- **Static Web App**: `stapp-<name>-<env>` or `swa-<name>`
- Names must be globally unique (use random suffix if needed)

## Bicep: Static Web App

```bicep
param staticWebAppName string
param location string = resourceGroup().location

resource swa 'Microsoft.Web/staticSites@2022-09-01' = {
  name: staticWebAppName
  location: location
  sku: {
    name: 'Free'   // or 'Standard' for custom domains, SLA
    tier: 'Free'
  }
  properties: {
    buildProperties: {
      appLocation: '/'
      outputLocation: 'dist'
      apiLocation: ''
    }
  }
}

output defaultHostname string = swa.properties.defaultHostname
output deploymentToken string = swa.listSecrets().properties.apiKey
```

## SKU Tiers

| Tier | Custom Domains | SLA | Staging | Auth |
|------|----------------|-----|---------|------|
| Free | No | No | Preview URLs only | Limited |
| Standard | Yes | Yes | Yes | Full |

## Build Configuration

Create `staticwebapp.config.json` at app root:

```json
{
  "routes": [
    {
      "route": "/api/*",
      "allowedRoles": ["authenticated"]
    },
    {
      "route": "/*",
      "serve": "/index.html",
      "statusCode": 200
    }
  ],
  "navigationFallback": {
    "rewrite": "/index.html"
  },
  "globalHeaders": {
    "X-Content-Type-Options": "nosniff"
  }
}
```

## GitHub Actions Deployment

1. Create Static Web App in Azure with source "Other" (or GitHub)
2. Copy deployment token to GitHub secret `AZURE_STATIC_WEB_APPS_API_TOKEN`
3. Use workflow:

```yaml
- uses: Azure/static-web-apps-deploy@v1
  with:
    azure_static_web_apps_api_token: ${{ secrets.AZURE_STATIC_WEB_APPS_API_TOKEN }}
    repo_token: ${{ secrets.GITHUB_TOKEN }}
    action: 'upload'
    app_location: '/'
    output_location: 'dist'
    api_location: ''   # or path to API (Azure Functions)
```

## Environment Variables (Build)

- `APP_LOCATION` – Client app folder
- `OUTPUT_LOCATION` – Build output (e.g., `dist`, `build`, `.next`)
- `API_LOCATION` – Optional API folder (Azure Functions)

## Framework-Specific Output

| Framework | output_location |
|-----------|-----------------|
| React (Vite) | `dist` |
| React (CRA) | `build` |
| Vue | `dist` |
| Angular | `dist/<project>/browser` |
| Svelte | `public` |
| Next.js (static) | `.next` or export to `out` |

## API (Optional)

Place Azure Functions in `api` folder; SWA serves them under `/api/*`. Supports JavaScript/TypeScript via Azure Functions runtime.

## Best Practices

1. **Use Standard tier** for production (custom domains, staging, SLA)
2. **Add `staticwebapp.config.json`** for SPA routing and headers
3. **Store deployment token** in GitHub/Azure DevOps secrets
4. **Use OIDC** instead of long-lived tokens when possible
5. **Enable staged previews** for PR validation
