---
name: azure-openai-llm
description: Use Azure OpenAI Service and LLMs on Azure. Use when working with Azure OpenAI, GPT-4, GPT-3.5, embeddings, chat completions, function calling, or when the user asks about hosting LLMs on Azure, Azure AI Studio, or integrating with OpenAI models in Azure.
---

# Azure OpenAI & LLMs on Azure Skill

## Quick Start

Azure OpenAI Service hosts OpenAI models (GPT-4, GPT-3.5, embeddings, etc.) on Azure with enterprise security and compliance.

## Resource & Deployment Model

1. **Create Azure OpenAI resource** (one per region/subscription)
2. **Deploy a model** (e.g., gpt-4o, gpt-4-turbo, gpt-35-turbo, text-embedding-ada-002)
3. **Call via REST or SDK** using endpoint + deployment name + API key or Azure AD

## Bicep: Azure OpenAI Resource + Deployment

```bicep
param cognitiveServicesName string
param deploymentName string = 'gpt-4o'
param modelName string = 'gpt-4o'
param modelVersion string = '2024-05-13'

resource openai 'Microsoft.CognitiveServices/accounts@2023-10-01-preview' = {
  name: cognitiveServicesName
  location: resourceGroup().location
  kind: 'OpenAI'
  sku: { name: 'S0' }
  properties: {}
}

resource deployment 'Microsoft.CognitiveServices/accounts/deployments@2023-10-01-preview' = {
  parent: openai
  name: deploymentName
  properties: {
    model: {
      format: 'OpenAI'
      name: modelName
      version: modelVersion
    }
    raiPolicyName: 'Microsoft.Default'
  }
}

output endpoint string = openai.properties.endpoint
output deploymentName string = deployment.name
```

## Authentication

| Method | Use Case |
|--------|----------|
| **API Key** | Quick dev; store in Key Vault for prod |
| **Azure AD / Managed Identity** | Production; use `DefaultAzureCredential` |

## Client Usage (Python)

```python
from openai import AzureOpenAI
from azure.identity import DefaultAzureCredential

# API Key
client = AzureOpenAI(
    api_key="<key>",
    api_version="2024-02-15-preview",
    azure_endpoint="https://<resource>.openai.azure.com"
)

# Azure AD (recommended for production)
credential = DefaultAzureCredential()
token = credential.get_token("https://cognitiveservices.azure.com/.default")
client = AzureOpenAI(
    api_key=token.token,
    api_version="2024-02-15-preview",
    azure_endpoint="https://<resource>.openai.azure.com"
)

response = client.chat.completions.create(
    model="gpt-4o",  # deployment name
    messages=[{"role": "user", "content": "Hello"}]
)
```

## Chat Completions

```python
response = client.chat.completions.create(
    model="gpt-4o",
    messages=[
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user", "content": "Explain ARM templates briefly."}
    ],
    temperature=0.7,
    max_tokens=500
)
```

## Function Calling

Define tools and let the model decide when to call them:

```python
tools = [{
    "type": "function",
    "function": {
        "name": "get_weather",
        "description": "Get weather for a location",
        "parameters": {
            "type": "object",
            "properties": {"location": {"type": "string"}},
            "required": ["location"]
        }
    }
}]
response = client.chat.completions.create(
    model="gpt-4o",
    messages=[{"role": "user", "content": "Weather in Seattle?"}],
    tools=tools,
    tool_choice="auto"
)
```

## Embeddings

```python
response = client.embeddings.create(
    model="text-embedding-ada-002",
    input="Your text here"
)
embedding = response.data[0].embedding
```

## Available Models (examples)

| Model | Deployment Name | Use Case |
|-------|-----------------|----------|
| gpt-4o | gpt-4o | Chat, vision, fast |
| gpt-4-turbo | gpt-4-turbo | Complex reasoning |
| gpt-35-turbo | gpt-35-turbo | Cost-effective chat |
| text-embedding-ada-002 | text-embedding-ada-002 | Embeddings |
| o1 | o1 | Advanced reasoning (preview) |

## Best Practices

1. **Use managed identity** or Azure AD instead of API keys in production
2. **Store endpoint and deployment name** in config (Key Vault or app settings)
3. **Set rate limits** (tokens per minute) in Azure OpenAI resource
4. **Use content filtering** (raiPolicyName) for safety
5. **Register for Azure OpenAI** if required in your subscription/region
