---
name: azure-devops-cicd
description: Create and configure Azure DevOps pipelines and GitHub Actions for Azure. Use when creating azure-pipelines.yml, GitHub Actions workflows for Azure, service connections, or when the user asks about deploying to Azure via CI/CD, Azure DevOps, or GitHub Actions.
---

# Azure DevOps & CI/CD Skill

## Quick Start

Azure deployments via CI/CD typically use:
- **Azure DevOps**: `azure-pipelines.yml`
- **GitHub Actions**: `.github/workflows/*.yml` with `azure/*` actions

## Azure DevOps Pipeline

### Service Connection

Create an Azure Resource Manager service connection (Service Principal) in Azure DevOps Project Settings → Service connections.

### Build & Deploy Template

```yaml
# azure-pipelines.yml
trigger:
  branches:
    include:
      - main

pool:
  vmImage: 'ubuntu-latest'

variables:
  azureSubscription: 'your-service-connection'
  webAppName: 'app-myapi-prod'
  resourceGroup: 'rg-myapi-prod'

stages:
  - stage: Build
    jobs:
      - job: Build
        steps:
          - task: NodeTool@0
            inputs:
              versionSpec: '18.x'
          - script: npm ci && npm run build
          - task: ArchiveFiles@2
            inputs:
              rootFolderOrFile: '$(System.DefaultWorkingDirectory)'
              includeRootFolder: false
              archiveType: 'zip'
              archiveFile: '$(Build.ArtifactStagingDirectory)/drop.zip'
          - task: PublishBuildArtifacts@1
            inputs:
              path: '$(Build.ArtifactStagingDirectory)'
              artifactName: 'drop'
              publishLocation: 'Container'

  - stage: Deploy
    dependsOn: Build
    jobs:
      - deployment: DeployWeb
        environment: 'production'
        strategy:
          runOnce:
            deploy:
              steps:
                - task: AzureWebApp@1
                  inputs:
                    azureSubscription: $(azureSubscription)
                    appType: 'webAppLinux'
                    appName: $(webAppName)
                    package: '$(Pipeline.Workspace)/drop/drop.zip'
```

## GitHub Actions

### Azure Login

```yaml
- uses: azure/login@v2
  with:
    creds: ${{ secrets.AZURE_CREDENTIALS }}
```

`AZURE_CREDENTIALS` JSON format:
```json
{
  "clientId": "<sp-app-id>",
  "clientSecret": "<sp-secret>",
  "subscriptionId": "<sub-id>",
  "tenantId": "<tenant-id>"
}
```

### Deploy Web App

```yaml
- uses: azure/webapps-deploy@v3
  with:
    app-name: ${{ env.AZURE_WEBAPP_NAME }}
    publish-profile: ${{ secrets.AZURE_WEBAPP_PUBLISH_PROFILE }}
    package: .
```

### Deploy Bicep/ARM

```yaml
- uses: azure/arm-deploy@v2
  with:
    subscriptionId: ${{ secrets.AZURE_SUBSCRIPTION_ID }}
    resourceGroupName: ${{ env.RESOURCE_GROUP }}
    template: ./infra/main.bicep
    parameters: environment=prod
```

## Best Practices

1. **Use service principals** (not user credentials) for CI/CD
2. **Store secrets** in Azure Key Vault or pipeline secret variables
3. **Use environments** (Azure DevOps) or environments (GitHub) for approvals
4. **Validate Bicep/ARM** in CI before deploy: `az bicep build` or `az deployment group validate`
5. **Use managed identity** on the build agent when running in Azure (e.g., Azure DevOps Microsoft-hosted or self-hosted in Azure)
