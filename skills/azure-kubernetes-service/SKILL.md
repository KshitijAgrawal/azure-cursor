---
name: azure-kubernetes-service
description: Create and manage Azure Kubernetes Service (AKS) clusters. Use when working with AKS, Kubernetes on Azure, kubectl, node pools, ingress, helm charts, or when the user asks about deploying containers to AKS, AKS Bicep/ARM, or cluster configuration.
---

# Azure Kubernetes Service (AKS) Skill

## Quick Start

AKS is a managed Kubernetes service. Use for container orchestration, scaling, and Kubernetes workloads on Azure.

## Naming Conventions

- **Cluster**: `aks-<name>-<env>` (e.g., `aks-myapp-prod`)
- **Node pool**: Often `system` and `user` pools; user pools can be named by workload
- **Resource group**: `rg-aks-<name>-<env>`

## Bicep: Basic AKS Cluster

```bicep
param clusterName string
param nodeCount int = 3
param vmSize string = 'Standard_D2s_v3'
param location string = resourceGroup().location

resource aks 'Microsoft.ContainerService/managedClusters@2024-02-01' = {
  name: clusterName
  location: location
  identity: {
    type: 'SystemAssigned'
  }
  properties: {
    dnsPrefix: clusterName
    agentPoolProfiles: [
      {
        name: 'systempool'
        count: nodeCount
        vmSize: vmSize
        mode: 'System'
        osType: 'Linux'
      }
    ]
    networkProfile: {
      networkPlugin: 'azure'
      loadBalancerSku: 'standard'
    }
  }
}

output kubeConfig string = aks.listClusterAdminCredential().kubeconfigs[0].value
output fqdn string = aks.properties.fqdn
```

## Node Pools

- **System pool**: Hosts critical system pods (CoreDNS, metrics-server). At least one node.
- **User pools**: Application workloads. Can add multiple for GPU, spot, or isolation.

```bicep
// Additional user pool in same cluster
{
  name: 'userpool'
  count: 2
  vmSize: 'Standard_D4s_v3'
  mode: 'User'
  minCount: 1
  maxCount: 5
  enableAutoScaling: true
}
```

## Authentication & Access

- **Azure AD integration**: Enable for RBAC and Azure AD groups
- **Managed identity**: Cluster identity for pulling images, Key Vault, etc.
- **kubectl**: Use `az aks get-credentials --resource-group <rg> --name <cluster>` to configure

## Ingress Options

| Option | Use Case |
|--------|----------|
| NGINX Ingress | Common, flexible |
| Application Gateway Ingress Controller (AGIC) | Native Azure LB, WAF |
| Application Gateway for Containers | Newer, HTTP routing |

## Common Commands

```bash
# Get kubeconfig
az aks get-credentials --resource-group <rg> --name <cluster>

# Create cluster (if not using IaC)
az aks create --resource-group <rg> --name <cluster> --node-count 3 --enable-managed-identity

# Scale node pool
az aks nodepool scale --resource-group <rg> --cluster-name <cluster> --name systempool --node-count 5

# Upgrade cluster
az aks upgrade --resource-group <rg> --name <cluster> --kubernetes-version <version>
```

## Best Practices

1. **Use managed identity** for the cluster (not service principal)
2. **Enable Azure AD** for production RBAC
3. **Use Azure CNI** or **Kubenet** based on VNet requirements
4. **Enable add-ons**: monitoring (Azure Monitor), HTTP application routing if needed
5. **Use separate node pools** for system vs application workloads
6. **Store secrets** in Azure Key Vault and use CSI driver or External Secrets Operator
