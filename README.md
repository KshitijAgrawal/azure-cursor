# Azure Cursor Plugin

A Cursor plugin to help developers integrate with Azure. It provides skills, rules, agents, and commands for ARM templates, Bicep, Azure Functions, App Service, Cosmos DB, CI/CD, and security best practices.

## Features

### Skills (AI capabilities)

| Skill | Use When |
|-------|----------|
| **arm-templates** | Creating or editing ARM JSON templates |
| **bicep** | Working with Bicep IaC (`.bicep` files) |
| **azure-functions** | Building or deploying Azure Functions |
| **azure-app-service** | Configuring or deploying Web Apps / APIs, slots, containers |
| **azure-static-web-apps** | Static sites, SPAs, JAMstack (React, Vue, Angular) |
| **azure-devops-cicd** | Setting up Azure DevOps or GitHub Actions pipelines |
| **cosmos-db** | Working with Cosmos DB (NoSQL, MongoDB, etc.) |
| **azure-storage** | Storage accounts, Blob, Queue, Table, File storage |
| **azure-kubernetes-service** | AKS clusters, kubectl, node pools, ingress |
| **azure-openai-llm** | Azure OpenAI, GPT-4, embeddings, chat completions |
| **azure-authentication** | Log in to Azure, az login, subscription selection |
| **azure-security-best-practices** | Microsoft Azure security patterns, Key Vault, RBAC, encryption |
| **azure-restricted-mode** | Read-only mode, restrict to resource group, safe testing |

### Rules

- **azure-naming** – Naming conventions for Azure resources
- **azure-security** – Security practices (Key Vault, managed identity, secrets)
- **azure-restricted-mode** – Enforce read-only or resource-group scope when configured

### Agents

- **azure-deployment-reviewer** – Reviews IaC and deployment configs for correctness and best practices

### Commands

- **deploy-azure** – Deploy apps or infrastructure to Azure
- **validate-iac** – Validate Bicep or ARM templates
- **generate-bicep** – Generate Bicep/ARM for a given Azure resource
- **azure-login** – Log in to Azure account for deployment

## Restricted Mode (Safe Testing)

You can restrict the plugin until you feel confident:

1. Copy `azure-cursor.config.json.example` to `azure-cursor.config.json` or `.cursor/azure-cursor.json` in your project root.
2. Set `restrictedMode: true` and choose a scope:
   - **`readOnly`** – No create/update/delete. Only login, list, validate.
   - **`resourceGroup`** – All writes limited to the `resourceGroup` you specify (e.g., `rg-my-test`).
3. When ready for full access, set `restrictedMode: false` or remove the config.

## Installation

Install from the [Cursor Marketplace](https://cursor.com/marketplace) or add the plugin locally:

1. Clone this repo
2. In Cursor: **Settings** → **Plugins** → **Add plugin from folder** → select this directory

## Usage

The plugin activates automatically when you work with Azure-related files or ask about:

- ARM templates, Bicep, `azuredeploy.json`
- Azure Functions, App Service, Cosmos DB
- Azure DevOps, GitHub Actions, CI/CD for Azure
- Azure naming, Key Vault, security

Example prompts:

- "Generate a Bicep template for an Azure Function App"
- "Review this ARM template for best practices"
- "Create an Azure DevOps pipeline to deploy a Node.js app"
- "How should I name my Key Vault and storage account?"

## Sample Walkthrough: Login, Deploy, and Cleanup

This walkthrough demonstrates logging in to Azure, creating a resource group, deploying a Storage Account via Bicep, and cleaning up when done. You can run these steps yourself or ask the Cursor AI (with this plugin) to execute them.

### Prerequisites

- [Azure CLI](https://learn.microsoft.com/azure/developer/cli/install-azure-cli) installed
- Azure subscription

### Step 1: Log in to Azure

```bash
az login
```

If you're on a headless/SSH session:

```bash
az login --use-device-code
```

Verify your account and set the subscription if needed:

```bash
az account show
az account set --subscription "<your-subscription-id-or-name>"
```

### Step 2: Create a resource group

```bash
LOCATION=eastus
RG_NAME=rg-azure-cursor-demo

az group create --name $RG_NAME --location $LOCATION
```

### Step 3: Deploy a resource (Storage Account) with Bicep

Create `main.bicep`:

```bicep
param location string = resourceGroup().location
param storageAccountName string = 'st${uniqueString(resourceGroup().id)}'

resource storageAccount 'Microsoft.Storage/storageAccounts@2023-01-01' = {
  name: storageAccountName
  location: location
  sku: { name: 'Standard_LRS' }
  kind: 'StorageV2'
}

output storageAccountName string = storageAccount.name
output storageAccountId string = storageAccount.id
```

Validate and deploy:

```bash
az bicep build --file main.bicep
az deployment group create \
  --resource-group $RG_NAME \
  --template-file main.bicep
```

Check the outputs:

```bash
az deployment group show --resource-group $RG_NAME --name main --query properties.outputs -o json
```

### Step 4: Clean up

Delete the resource group (and everything inside it):

```bash
az group delete --name $RG_NAME --yes --no-wait
```

Use `--no-wait` to return immediately; the delete runs in the background. Omit it to wait for completion.

Verify deletion:

```bash
az group exists --name $RG_NAME
# Returns: false
```

### Using the plugin with Cursor

You can drive this flow via natural language in Cursor:

- *"Log in to my Azure account"* → triggers azure-authentication
- *"Create a resource group named rg-demo in eastus"* → agent runs the command
- *"Generate a Bicep template for a Storage Account and deploy it to rg-demo"* → agent creates Bicep and runs deployment
- *"Delete the rg-demo resource group and all resources"* → agent runs cleanup

## Structure

```
azure-cursor/
├── .cursor-plugin/
│   └── plugin.json
├── skills/
│   ├── arm-templates/
│   ├── bicep/
│   ├── azure-functions/
│   ├── azure-app-service/
│   ├── azure-static-web-apps/
│   ├── azure-devops-cicd/
│   ├── cosmos-db/
│   ├── azure-storage/
│   ├── azure-kubernetes-service/
│   ├── azure-openai-llm/
│   ├── azure-authentication/
│   ├── azure-restricted-mode/
│   └── azure-security-best-practices/
├── rules/
│   ├── azure-naming.mdc
│   ├── azure-security.mdc
│   └── azure-restricted-mode.mdc
├── agents/
│   └── azure-deployment-reviewer.md
├── commands/
│   ├── deploy-azure.md
│   ├── validate-iac.md
│   ├── generate-bicep.md
│   └── azure-login.md
├── docs/
│   └── PLUGIN_PLAN.md
├── azure-cursor.config.json.example
├── azure-cursor.config.schema.json
└── README.md
```

## License

MIT
