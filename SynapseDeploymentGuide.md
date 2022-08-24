# Synapse Analytics Overview 
### The following links below can be used as references for additional information within the deployment guide, if needed.

Pre-requsites - 

[Configure IP firewall rules - Azure Synapse Analytics | Microsoft Docs]
[Managed virtual network - Azure Synapse Analytics | Microsoft Docs
Managed private endpoints - Azure Synapse Analytics | Microsoft Docs
Data exfiltration protection for Azure Synapse Analytics workspaces - Azure Synapse Analytics | Microsoft Docs
Connect to a Synapse Studio using private links - Azure Synapse Analytics | Microsoft Docs
Authorize server and database access using logins and user accounts - Azure SQL Database & SQL Managed Instance & Azure Synapse Analytics | Microsoft Docs
Azure Active Directory - Azure Synapse Analytics | Microsoft Docs
Azure Synapse workspace access control overview - Azure Synapse Analytics | Microsoft Docs
Azure Synapse Analytics security white paper - Azure Synapse Analytics | Microsoft Docs]


## Deployment Guide for Synapse Analytics
### This guide has been put together with the intent of assisting customers deploly Synapse within IL5. Each section is broken out by the following- 

  - Pre-requsites needed
      - Firewall
      - Vnet
      - Active Directory
      - Azure Key Vault
  - Deployment steps
      - SQL Pool Configuration
      - Permissions
      - Private Endpoint Configuration
  - Validation
      - Accessiblity
      - Scripting
      - Create a pipeline (optional)

Pre-requisites:
  - Setup proper Vnet and ensure proper subnet\IP ranges are established:
      - A Vnet, or specific subnet spaces will need to be carved out to support the deployment of Synapse. When planning to deploy to an IL5 environment, all private endpoints within Synapse must be secured when connectivity between Synapse and other services i.e. storage account(s), are being established.
  - Ensure Active directory is setup within your respective environment and identity is able to sync and communicate with Synapse and any other resource.
      - Having the capability within Synapse to separate\segregate user access via Active Directory provides a more controlled and secured approach in how this resource is accessed.
      - Specific service accounts within AD will also need to be leveraged when configuring Synapse to ensure secure access control throughout the resource is in place.
      - Being able to secure user accounts via MFA provides an extra layer of security.
   - Ensure Azure Key Vault is stood up and setup to integrate properly.
      - Azure Key Vault is necessary for any type of certificate, credential, or secret management that is required throughout Synapse.

