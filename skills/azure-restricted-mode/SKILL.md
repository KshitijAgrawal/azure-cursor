---
name: azure-restricted-mode
description: Understand and configure Azure Cursor restricted mode. Use when the user wants read-only mode, to restrict operations to a resource group, or to safely test the plugin before full access.
---

# Azure Restricted Mode Skill

## Overview

Restricted mode lets users safely try the plugin with limited impact:

- **readOnly**: No create/update/delete—only login, list, validate
- **resourceGroup**: All writes limited to one resource group (e.g., for testing)

## Configuration

Create `azure-cursor.config.json` or `.cursor/azure-cursor.json` in the project root:

**Read-only mode** (no deployments, no resource creation):
```json
{
  "restrictedMode": true,
  "scope": "readOnly"
}
```

**Resource-group-scoped mode** (only operate in one RG):
```json
{
  "restrictedMode": true,
  "scope": "resourceGroup",
  "resourceGroup": "rg-my-test"
}
```

**Disable restrictions**:
```json
{
  "restrictedMode": false
}
```

## Behavior

| Scope | Allowed | Blocked |
|-------|---------|---------|
| readOnly | az login, az account show, az group list, az resource list, az bicep build, az deployment group validate | az group create/delete, az deployment group create, any create/update/delete |
| resourceGroup | Writes only to the configured resource group | Writes to other resource groups |

## Setup

1. Copy `azure-cursor.config.json.example` from the plugin to your project root.
2. Rename to `azure-cursor.config.json`.
3. Set `restrictedMode: true` and choose `scope`.
4. When confident, set `restrictedMode: false` or remove the config.
