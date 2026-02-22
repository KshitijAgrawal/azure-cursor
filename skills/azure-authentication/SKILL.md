---
name: azure-authentication
description: Log in to Azure and manage authentication. Use when the user needs to sign in to Azure, run az login, configure Azure CLI, select subscription, verify login, or when deploying Azure resources requires authentication setup.
---

# Azure Authentication Skill

## Quick Start

Before deploying or managing Azure resources, authenticate with Azure. The Azure CLI (`az`) is the primary tool for login and deployment.

## Interactive Login (Local Development)

Opens a browser for sign-in. Use for local development and one-off operations:

```bash
az login
```

Outputs subscription info. Default subscription is used for subsequent commands.

## Verify Current Login

```bash
# Show current account and subscription
az account show

# List all subscriptions
az account list --output table

# Set active subscription
az account set --subscription "<subscription-id-or-name>"
```

## Device Code (Headless / SSH)

When browser is unavailable (SSH, remote sessions, some CI):

```bash
az login --use-device-code
```

Follow the URL and code printed in the terminal.

## Service Principal (CI/CD, Scripts)

For automation, use a service principal. **Never commit credentials to source control.**

```bash
az login --service-principal \
  --username $ARM_CLIENT_ID \
  --password $ARM_CLIENT_SECRET \
  --tenant $ARM_TENANT_ID
```

Common env var names (Terraform/Azure RM): `ARM_CLIENT_ID`, `ARM_CLIENT_SECRET`, `ARM_TENANT_ID`, `ARM_SUBSCRIPTION_ID`.

Create a service principal:

```bash
az ad sp create-for-rbac --name "my-sp" --role contributor \
  --scopes /subscriptions/<subscription-id>
```

## Azure Developer CLI (azd)

If using Azure Developer CLI for full-stack deployments:

```bash
azd auth login
```

Uses browser flow. After login, `azd up` and `azd deploy` work against the authenticated tenant.

## Logout

```bash
az logout
```

## Troubleshooting

| Issue | Action |
|-------|--------|
| "Please run 'az login'" | Run `az login` or `az login --use-device-code` |
| Wrong subscription | `az account set --subscription <id>` |
| Token expired | Run `az login` again |
| SP auth fails | Verify `ARM_CLIENT_ID`, `ARM_CLIENT_SECRET`, `ARM_TENANT_ID`; check SP has Contributor (or appropriate) role on scope |

## Security Best Practices

1. **Use interactive login** for local dev; avoid storing credentials in files
2. **Use service principals** in CI/CD; store secrets in pipeline variables or Key Vault
3. **Scope service principals** to resource groups, not subscription, when possible
4. **Prefer managed identity** when code runs in Azure (VM, App Service, Function, etc.)
