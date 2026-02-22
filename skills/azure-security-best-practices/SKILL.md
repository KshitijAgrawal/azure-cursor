---
name: azure-security-best-practices
description: Apply Microsoft Azure security best practices. Use when reviewing or designing Azure deployments, IaC, or when the user asks about Azure security, Key Vault, managed identity, RBAC, encryption, or network security.
---

# Azure Security Best Practices Skill

Based on [Microsoft Azure security best practices and patterns](https://learn.microsoft.com/en-us/azure/security/fundamentals/best-practices-and-patterns).

## Quick Reference

### Secrets & Key Vault
- Never hardcode secrets; use Key Vault references in App Service, Functions.
- Use `securestring` / `@secure()` in ARM/Bicep for sensitive parameters.
- Managed Identity over connection strings when supported.
- Rotate secrets; revoke immediately if exposed.

### Identity & Access
- RBAC with least privilege; smallest scope (resource group, resource).
- Managed Identity for App Service, Functions, AKS, VMs.
- Enforce MFA for admins; avoid long-lived service principal secrets in CI/CD.

### Data
- TLS 1.2+ minimum; `supportsHttpsTrafficOnly: true` on Storage.
- `allowBlobPublicAccess: false` unless required.
- Customer-managed keys (CMK) for stricter control when needed.

### Network
- Private endpoints for Key Vault, Storage, SQL, Cosmos DB in production.
- VNet integration for App Service/Functions when backend is internal.

### IaC & Deployment
- No secrets in templates; Key Vault references or parameter files (not in repo).
- Validate templates before deploy; use deployment slots for zero-downtime.
- Store pipeline secrets in Key Vault or pipeline variables.

## When Reviewing IaC

Check: Key Vault refs, managed identity, TLS settings, no public blob access, appropriate RBAC, tags.

## When Reviewing CI/CD

Check: Secrets in variables/Key Vault, no creds in YAML, use OIDC or short-lived tokens.

## Reference

For detailed guidance, see [references/microsoft-security-patterns.md](references/microsoft-security-patterns.md).
