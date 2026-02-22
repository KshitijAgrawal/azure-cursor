# Microsoft Azure Security Best Practices (Reference)

Source: [Azure security best practices and patterns](https://learn.microsoft.com/en-us/azure/security/fundamentals/best-practices-and-patterns)

## Secrets (Key Vault)

- **Never hardcode secrets** in code, config, or IaC. Use Key Vault references.
- **Rotate secrets regularly**; automate where possible. Revoke immediately if compromised.
- **Use Managed Identity** for app-to-Azure auth instead of connection strings.
- **Apply least privilege** via RBAC; avoid broad `*` permissions on Key Vault.
- **Enable logging** for Key Vault; monitor access via Azure Monitor.
- **Network isolation**: Use firewall/NSG, private endpoints for Key Vault in production.
- **Encrypt at rest and in transit**: Key Vault uses envelope encryption; use HTTPS for API calls.

## Identity & Access (Entra ID)

- **Treat identity as primary perimeter**. Use Microsoft Entra ID (Azure AD) for auth.
- **Use RBAC** for Azure resources; assign roles at the smallest scope needed.
- **Lower exposure of privileged accounts**; use PIM (Privileged Identity Management) for just-in-time access.
- **Enforce MFA** for users, especially admins.
- **Managed identities** for workloads: App Service, Functions, AKS, VMs.
- **Block legacy auth** via Conditional Access where possible.

## Data Protection

- **At rest**: Azure Storage, SQL, Cosmos DB encrypt by default. Use customer-managed keys (CMK) for stricter control.
- **In transit**: Use TLS 1.2+; enable HTTPS only on storage; no plain HTTP.
- **Minimum TLS version**: Set `minimumTlsVersion` to TLS1_2 on Storage, Key Vault.
- **Disable public access** on Storage blobs unless explicitly required.

## Network Security

- **Private endpoints** for Key Vault, Storage, SQL, Cosmos DB in production.
- **NSGs** on subnets; restrict traffic to required ports.
- **Avoid public IPs** for backend services; use VNet integration for App Service.
- **Use Azure VPN or ExpressRoute** for hybrid connectivity.

## Operational Security

- **Enable diagnostic logs** for App Service, Key Vault, Storage; send to Log Analytics.
- **Application Insights** for app monitoring and anomaly detection.
- **Secure management workstations** for subscription/tenant admins.

## Service-Specific Links

- [Secrets best practices](https://learn.microsoft.com/en-us/azure/security/fundamentals/secrets-best-practices)
- [Identity management best practices](https://learn.microsoft.com/en-us/azure/security/fundamentals/identity-management-best-practices)
- [Data encryption best practices](https://learn.microsoft.com/en-us/azure/security/fundamentals/data-encryption-best-practices)
- [Network best practices](https://learn.microsoft.com/en-us/azure/security/fundamentals/network-best-practices)
- [PaaS best practices](https://learn.microsoft.com/en-us/azure/security/fundamentals/paas-deployments)
- [AI security best practices](https://learn.microsoft.com/en-us/azure/security/fundamentals/ai-security-best-practices)
