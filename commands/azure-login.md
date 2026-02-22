---
name: azure-login
description: Log in to Azure account for deployment and resource management
---

# Log In to Azure

When the user needs to log in to Azure before deploying or managing resources. Login and account verification are always allowed, even in restricted mode.

1. **Interactive (local)**: Run `az login` to open browser. Works when user has GUI/browser.
2. **Headless / device code**: Run `az login --use-device-code` when no browser (SSH, remote).
3. **Verify**: Run `az account show` to confirm login and see active subscription.
4. **Change subscription**: Run `az account set --subscription "<id-or-name>"` if needed.
5. **CI/CD**: Use service principal with env vars `ARM_CLIENT_ID`, `ARM_CLIENT_SECRET`, `ARM_TENANT_ID`. Reference azure-authentication skill for setup.

After login, deployment commands (Bicep/ARM deploy, webapp deploy, etc.) will succeed.
