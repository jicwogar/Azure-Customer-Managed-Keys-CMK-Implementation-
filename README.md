# Azure Customer-Managed Keys (CMK) Implementation

## Project Overview

This project demonstrates how to implement **Customer-Managed Keys (CMK)** for an Azure Storage Account using **Azure Key Vault** and a **System-Assigned Managed Identity**.

The implementation provides a practical example of controlling encryption keys independently of Microsoft-managed keys. It demonstrates how Azure Key Vault can be used to securely store an RSA key and how an Azure Storage Account can use that key for server-side encryption.

The project also demonstrates the configuration of the required Azure RBAC permissions that allow the Storage Account's managed identity to access and use the encryption key.

## Objectives

The primary objectives of this project are to:

- Create the required Azure resource group and Storage Account.
- Create and configure an Azure Key Vault.
- Create an RSA encryption key in Key Vault.
- Configure a System-Assigned Managed Identity for the Storage Account.
- Obtain the Storage Account's managed identity principal ID.
- Assign the required Key Vault RBAC permissions.
- Configure Customer-Managed Key encryption on the Storage Account.
- Verify that the Storage Account is using the Customer-Managed Key.
- Demonstrate the implementation using Azure CLI, Azure Portal, and ARM template concepts.

## Architecture

```text
                    Azure Subscription
                           |
                           v
                    Resource Group
                           |
              +------------+-------------+
              |                          |
              v                          v
       Azure Storage Account        Azure Key Vault
              |                          |
              | System-Assigned          | RSA Key
              | Managed Identity         |
              |                          |
              +---------> RBAC ----------+
                           |
                           v
                  Customer-Managed Key
                           |
                           v
                 Storage Data Encryption
```

## Technologies Used

| Technology | Purpose |
|---|---|
| Microsoft Azure | Cloud infrastructure platform |
| Azure Storage Account | Data storage resource |
| Azure Key Vault | Secure storage and management of encryption keys |
| Customer-Managed Key (CMK) | Customer-controlled encryption key |
| Managed Identity | Passwordless identity for Azure resources |
| Azure RBAC | Authorization to access and use the Key Vault key |
| Azure CLI | Resource configuration and administration |
| ARM Template | Infrastructure-as-Code deployment |
| Azure Portal | Configuration and verification |

## Prerequisites

Before starting the implementation, ensure you have:

- An active Azure subscription.
- Permission to create and manage Azure resources.
- Permission to assign Azure RBAC roles.
- Access to Azure Portal and/or Azure Cloud Shell.
- Basic knowledge of Azure Storage, Key Vault, managed identities, and RBAC.

---

# Implementation Steps

## Step 1 — Create the Resource Group

Create a dedicated resource group to contain the resources used by the CMK implementation.

Example Azure CLI command:

```bash
az group create   --name <resource-group-name>   --location <azure-region>
```

A dedicated resource group makes it easier to manage, monitor, and remove the resources associated with the project.

!Step 1. create a resource gruop using  bash in cloud shell](Step%201.%20create%20a%20resource%20group%20using%20bash%20in%20cloud%20shell.png)

---

## Step 2 — Create the Storage Account

Create the Azure Storage Account that will eventually use the Customer-Managed Key.

The Storage Account should support encryption with customer-managed keys and should be configured according to the workload's security and availability requirements.

![Step 2 - Create storage account](Step%202.%20create%20a%20storage%20account.png)

---

## Step 3 — Create the Key Vault

Create an Azure Key Vault to securely store the encryption key.

The Key Vault provides centralized key management and allows access to encryption keys to be controlled using Azure RBAC.

![Step 3 - Create Key Vault](Step%203.%20create%20a%20vault%20in%20my%20resource%20group.png)

---

## Step 4 — Grant Account Permissions

Grant the required permissions to the account performing the configuration.

Depending on the selected Key Vault authorization model, this may involve assigning an appropriate Azure RBAC role at the Key Vault scope.

The administrator must have sufficient permissions to create keys and configure the Storage Account's encryption settings.

![Step 4 - Grant account permissions](Step%204.%20Grant%20Your%20account%20to%20create.png)

---

## Step 5 — Create an RSA Key

Create an RSA key in Azure Key Vault.

The key becomes the Customer-Managed Key that Azure Storage will use for encryption operations.

Example:

```bash
az keyvault key create   --vault-name <key-vault-name>   --name <key-name>   --kty RSA
```

![Step 5 - Create RSA key](Step%206.%20create%20an%20RSA%20key.png)

---

## Step 6 — Configure the Storage Account Managed Identity

Enable a **System-Assigned Managed Identity** on the Storage Account.

This identity allows Azure Storage to authenticate to Key Vault without storing credentials, passwords, or secrets in the application or deployment configuration.

Example:

```bash
az storage account update   --name <storage-account-name>   --resource-group <resource-group-name>   --assign-identity
```

![Step 7 - Assign Key Vault role to Storage Identity](Step%207.%20assign%20key%20vault%20role%20to%20storage%20identity.png)

---

## Step 7 — Obtain the Managed Identity Principal ID

Retrieve the Storage Account's managed identity principal ID.

This identifier is required when assigning Key Vault RBAC permissions to the Storage Account.

Example:

```bash
az storage account show   --name <storage-account-name>   --resource-group <resource-group-name>   --query identity.principalId   --output tsv
```

![Step 8 - Obtain identity principal ID](Step%208.%20obtain%20identity%20principal%20id.png)

---

## Step 8 — Assign the Required Key Vault RBAC Role

Assign the appropriate Key Vault role to the Storage Account's managed identity.

The role must allow Azure Storage to perform the key operations required for customer-managed encryption.

The exact role should be selected according to Microsoft's current Azure Storage CMK requirements and the organization's least-privilege policy.

Example pattern:

```bash
az role assignment create   --assignee <storage-managed-identity-principal-id>   --role "<required-key-vault-role>"   --scope <key-vault-resource-id>
```

![Step 9 - Assign Key Vault role](Step%209.%20assign%20key%20vault%20role%20to%20storage%20identity.png)

---

## Step 9 — Configure Customer-Managed Key Encryption

Configure the Storage Account to use the Key Vault RSA key for encryption.

The configuration associates:

- Storage Account
- Key Vault
- Key
- Managed Identity

This establishes the trust relationship required for Customer-Managed Key encryption.

![Step 10 - Enable customer-managed key encryption](Step%2010.%20enable%20customer%20managed%20key%20encryption.png)

---

## Step 10 — Assign Required Service Identity Permissions

Verify that the appropriate identity and role assignments are in place.

The Storage Account must be able to access the Key Vault key when Azure performs encryption-related operations.

![Step 11 - Assign service identity role](Step%2011.%20assigning%20service%20id%20and%20role%20assignment.png)

---

## Step 11 — Verify the Key Vault Configuration

Verify that the Key Vault and encryption key are correctly configured.

Check:

- Key Vault exists and is accessible.
- RSA key exists.
- Key is enabled.
- Required RBAC assignments exist.
- Storage Account managed identity exists.
- Storage Account has access to the key.

---

## Step 12 — Final Verification

Perform a final verification of the Microsoft Key Vault configuration and Storage Account encryption settings.

The objective is to confirm that the Storage Account is configured to use the intended Customer-Managed Key rather than relying solely on Microsoft-managed encryption keys.

![Final verification](Step%2012.%20final%20step.%20verify%20microsoft%20key%20vault.png)

---

# Security Model

The implementation follows a **managed identity + RBAC + Key Vault** security model.

```text
Storage Account
      |
      | System-Assigned Managed Identity
      v
Azure RBAC
      |
      | Authorized Key Operations
      v
Azure Key Vault
      |
      v
Customer-Managed RSA Key
      |
      v
Storage Encryption
```

This approach avoids embedding credentials in scripts or applications and provides centralized control over encryption keys.

## Why Customer-Managed Keys?

Azure Storage encrypts data at rest by default. Customer-Managed Keys provide additional control over the encryption key lifecycle.

CMK can be useful when an organization requires:

- Greater control over encryption keys.
- Centralized key management.
- Key rotation management.
- Ability to disable or revoke key access.
- Compliance with organizational or regulatory requirements.
- Separation of data ownership from cloud-provider-managed keys.

## Managed Identity Benefits

The Storage Account's System-Assigned Managed Identity provides:

- Passwordless authentication.
- No credentials stored in application code.
- Automatic identity lifecycle management.
- Integration with Azure RBAC.
- Reduced risk associated with secrets and credentials.

## Azure RBAC

RBAC is used to grant the Storage Account identity only the permissions required to access the encryption key.

This supports the **principle of least privilege**.

---

# Key Management Considerations

Customer-managed encryption introduces additional operational responsibilities.

Administrators should plan for:

### Key Rotation

Establish a controlled key-rotation process and understand how new key versions are handled.

### Key Availability

The Storage Account depends on access to the Key Vault key. Key access or Key Vault availability issues can affect encryption-related operations.

### Key Disablement

Disabling or deleting a key can have significant consequences. Key lifecycle operations should therefore be tightly controlled.

### Soft Delete and Purge Protection

Key Vault should be configured with appropriate protection against accidental deletion and unauthorized permanent removal of keys.

### Monitoring

Monitor Key Vault and Storage Account activity using Azure Monitor, diagnostic settings, and relevant security alerts.

---

# Infrastructure as Code

An ARM template is included in the repository to support repeatable deployment of the CMK configuration.

Infrastructure as Code provides several advantages:

- Repeatable deployments
- Consistent configuration
- Version control
- Easier auditing
- Reduced manual configuration
- Improved disaster recovery and environment replication

The ARM template can be reviewed and adapted for deployment into development, test, or production environments.

---

# Validation Checklist

- [x] Resource group created.
- [x] Azure Storage Account created.
- [x] Azure Key Vault created.
- [x] RSA encryption key created.
- [x] Storage Account managed identity configured.
- [x] Managed identity principal ID obtained.
- [x] Key Vault RBAC permissions assigned.
- [x] Customer-Managed Key encryption configured.
- [x] Key Vault configuration verified.
- [x] Final encryption configuration verified.
- [x] Implementation documented with screenshots.
- [x] ARM template included for repeatable deployment.

---

# Skills Demonstrated

This project demonstrates practical experience with:

- Azure Storage security
- Customer-Managed Keys
- Azure Key Vault
- Encryption at rest
- Managed identities
- Azure RBAC
- Azure CLI
- Azure Portal
- ARM templates
- Infrastructure as Code
- Cloud security architecture
- Least-privilege access control
- Key lifecycle management

---

# Production Considerations

For a production deployment, the following additional controls should be considered:

- Use a dedicated Key Vault with appropriate administrative boundaries.
- Enable Key Vault soft delete and purge protection.
- Apply least-privilege Azure RBAC.
- Restrict Key Vault network access where appropriate.
- Consider Private Endpoints for sensitive environments.
- Monitor Key Vault access and key operations.
- Implement key rotation procedures.
- Document key-recovery procedures.
- Use Azure Policy to enforce encryption and security standards.
- Store infrastructure code in source control.
- Use separate keys and Key Vaults where appropriate for environment isolation.
- Test key rotation and recovery procedures before production deployment.

---

# Project Outcome

The completed implementation demonstrates how an Azure Storage Account can be configured to use an **Azure Key Vault Customer-Managed Key** through a **System-Assigned Managed Identity** and **Azure RBAC**.

The project provides practical evidence of understanding of Azure encryption architecture, identity-based access control, key management, and Infrastructure as Code.

---

## Repository Structure

```text
Azure-Customer-Managed-Keys-CMK-Implementation/
│
├── README.md
├── arm template for Az CMK Implementation
│
├── Step 1. create a resource group...
├── Step 2. create a storage account.png
├── Step 3. create a vault...
├── Step 4. Grant Your account...
├── Step 6. create an RSA key.png
├── Step 7. assign key vault role...
├── Step 8. obtain identity principal id.png
├── Step 9. assign key vault role...
├── Step 10. enable customer managed key encryption...
├── Step 11. assigning service id...
└── Step 12. final step...
```

---

**Author:** J. C. Wogar  
**Platform:** Microsoft Azure  
**Project:** Azure Customer-Managed Keys (CMK) Implementation
