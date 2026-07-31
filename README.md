Azure Customer-Managed Keys (CMK) Implementation
This project demonstrates how to configure Customer-Managed Keys (CMK) for an Azure Storage Account using Azure Key Vault and a System-Assigned Managed Identity. The goal is to improve the security of data at rest by allowing the organization to control the encryption keys instead of relying solely on Microsoft-managed keys.
📌 Architecture Overview
Azure Key Vault (erckeyvault01) was created as the secure location for storing encryption keys. Both Soft Delete and Purge Protection were enabled to help prevent accidental or malicious deletion of keys.
An RSA 2048-bit encryption key named cmk-storage-key was generated in the Key Vault. Key management was performed using Azure RBAC with the Key Vault Crypto Officer role.
The Azure Storage Account (ercmslearnstorage) was configured with a System-Assigned Managed Identity. This identity was granted the Key Vault Crypto Service Encryption User role, allowing the storage account to securely access the encryption key without storing credentials.
Finally, the storage account was configured to use the customer-managed key by referencing the Key Vault URI and key name. Azure Storage performs envelope encryption, using Microsoft.KeyVault as the key source to protect data stored at rest.

Step-by-Step CLI Execution

1. Register the Key Vault Resource Provider

Before creating Azure Key Vault resources, make sure the Microsoft.KeyVault resource provider is registered for your Azure subscription. If it is already registered, running the command again has no adverse effect.

az provider register --namespace Microsoft.KeyVault
2. Provision Azure Key Vault
​Deploy Key Vault with Purge Protection enabled (Soft Delete is on by default):

2. Create the Azure Key Vault

Create an Azure Key Vault to securely store the customer-managed encryption key. Purge Protection is enabled during creation to prevent the key from being permanently deleted, while Soft Delete is enabled by default for all new Key Vaults.

az keyvault create \
  --name erckeyvault01 \
  --resource-group ErcMslearngroup \
  --location eastasia \
  --enable-purge-protection true
  
  3. Configure Data Plane RBAC Permissions

Grant your signed-in user account the required Azure RBAC role to create and manage cryptographic keys in Azure Key Vault. Use your Azure AD (Microsoft Entra ID) User Object ID when assigning the role.

Run the following commands to retrieve your signed-in user's Microsoft Entra ID Object ID and assign the Key Vault Crypto Officer role. This role allows your account to create and manage cryptographic keys within the Azure Key Vault.

# Get the signed-in user's Object ID
USER_OID=$(az ad signed-in-user show --query id -o tsv)

# Assign the Key Vault Crypto Officer role
az role assignment create \
  --assignee-object-id $USER_OID \
  --assignee-principal-type User \
  --role "Key Vault Crypto Officer" \
  --scope $(az keyvault show --name erckeyvault01 --query id -o tsv)
  
  4. Create the Cryptographic Key

Generate an RSA 2048-bit encryption key in Azure Key Vault. This key will be used as the Customer-Managed Key (CMK) for encrypting data stored in the Azure Storage Account.

az keyvault key create \
  --vault-name erckeyvault01 \
  --name cmk-storage-key \
  --kty RSA \
  --size 2048
  
  5. Configure the Storage Account Managed Identity

Enable a System-Assigned Managed Identity on the target Azure Storage Account ("ercmslearnstorage"). This managed identity allows the storage account to securely authenticate with Azure Key Vault without requiring stored credentials or secrets.

az storage account update \
  --name ercmslearnstorage \
  --resource-group ErcMslearngroup \
  --assign-identity
  
  6. Grant the Storage Account Access to the Key Vault

Grant the Storage Account's System-Assigned Managed Identity permission to use the customer-managed key stored in Azure Key Vault. Assign the Key Vault Crypto Service Encryption User role to the storage account's Principal ID so it can perform the cryptographic operations required for encryption and decryption.

# Get the Storage Account's Principal ID
STORAGE_PRINCIPAL_ID=$(az storage account show \
  --name ercmslearnstorage \
  --resource-group ErcMslearngroup \
  --query identity.principalId \
  -o tsv)

# Assign the Key Vault Crypto Service Encryption User role

STORAGE_PID=$(az storage account show --name ercmslearnstorage --resource-group ErcMslearngroup --query identity.principalId -o tsv)

az role assignment create \
  --assignee-object-id $STORAGE_PID \
  --assignee-principal-type ServicePrincipal \
  --role "Key Vault Crypto Service Encryption User" \
  --scope $(az keyvault show --name erckeyvault01 --query id -o tsv)
  7. Enable Customer-Managed Key Encryption
​Link the Key Vault key to the storage account to encrypt data at rest:

az storage account update \
  --name ercmslearnstorage \
  --resource-group ErcMslearngroup \
  --encryption-key-name cmk-storage-key \
  --encryption-key-vault https://erckeyvault01.vault.azure.net/ \
  --encryption-key-source Microsoft.Keyvault

  Verification
​Verify that the storage account is actively using Customer-Managed Keys:
az storage account show \
  --name ercmslearnstorage \
  --resource-group ErcMslearngroup \
  --query encryption.keySource -o tsv

