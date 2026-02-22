# Azure Cursor Plugin – Planning Document

## Overview

This Cursor plugin helps developers integrate with Azure by providing skills, rules, agents, and commands for the most-used Azure services and workflows.

## Most Used Azure Functionalities (Prioritized)

### Tier 1: Core Compute & Hosting
| Service | Use Case | Plugin Support |
|---------|----------|----------------|
| **Azure App Service** | Web apps, APIs (.NET, Java, Node.js, Python) | Skill, rules |
| **Azure Functions** | Serverless, event-driven workloads | Skill |
| **Azure Static Web Apps** | Static sites, JAMstack, React/Vue/Angular | Skill, rules |
| **Azure Container Apps** | Microservices, containers | References in skills |

### Tier 2: Infrastructure as Code
| Tool | Use Case | Plugin Support |
|------|----------|----------------|
| **ARM Templates** | JSON-based IaC, legacy/new deployments | Dedicated skill |
| **Bicep** | Modern Azure IaC, transpiles to ARM | Dedicated skill |

### Tier 3: Data & Storage
| Service | Use Case | Plugin Support |
|---------|----------|----------------|
| **Azure SQL Database** | Managed SQL Server | Patterns in skills |
| **Cosmos DB** | NoSQL, multi-model, global distribution | Skill references |
| **Storage (Blob, Queue, Table)** | Object storage, messaging | Patterns in skills |

### Tier 4: Security & Identity
| Service | Use Case | Plugin Support |
|---------|----------|----------------|
| **Azure Key Vault** | Secrets, certs, keys | Rules, skill references |
| **Managed Identity** | Passwordless auth | Rules |

### Tier 5: DevOps & CI/CD
| Service | Use Case | Plugin Support |
|---------|----------|----------------|
| **Azure DevOps** | Pipelines, repos, boards | Skill |
| **GitHub Actions (Azure)** | CI/CD with Azure | Skill references |

### Tier 6: AI & Integration
| Service | Use Case | Plugin Support |
|---------|----------|----------------|
| **Azure OpenAI** | LLM APIs | Skill references |
| **Azure AI Services** | Vision, Speech, Language | Skill references |

## Plugin Components

### Skills
- **arm-templates** – Author and validate ARM JSON templates
- **bicep** – Author and validate Bicep IaC
- **azure-functions** – Create and deploy Azure Functions
- **azure-app-service** – Configure and deploy App Service (Web Apps, slots, containers)
- **azure-static-web-apps** – Static sites, SPAs, JAMstack
- **azure-devops-cicd** – Pipelines, service connections, deployments
- **cosmos-db** – Cosmos DB, partition keys, throughput
- **azure-storage** – Storage accounts, Blob, Queue, Table, File, connection strings
- **azure-kubernetes-service** – AKS clusters, node pools, ingress, kubectl
- **azure-openai-llm** – Azure OpenAI, GPT-4, embeddings, chat completions, function calling
- **azure-authentication** – Log in to Azure (az login, device code, service principal), subscription selection
- **azure-restricted-mode** – Read-only or resource-group-scoped mode for safe testing
- **azure-security-best-practices** – Microsoft security patterns (secrets, identity, data, network)

### Rules
- Azure naming conventions
- Security best practices (Key Vault, managed identity)
- ARM/Bicep style and structure

### Agents
- Azure deployment reviewer
- Azure security reviewer

### Commands
- Deploy to Azure
- Generate ARM/Bicep for resource
- Validate IaC templates

## Skill Triggers

Each skill uses specific trigger terms in its `description` so the agent knows when to apply it:

- ARM: "ARM template", "arm template", "azure template", "azure resource manager"
- Bicep: "bicep", ".bicep", "Azure IaC"
- Functions: "Azure Functions", "function app", "serverless"
- App Service: "App Service", "web app", "azure web app"
- DevOps: "Azure DevOps", "YAML pipeline", "azure pipeline"
