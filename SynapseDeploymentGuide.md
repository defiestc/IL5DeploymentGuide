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



Step 1- Login to portal.azure.gov and select *+Create Resource* in the top left of the screen.
![image](https://user-images.githubusercontent.com/95705084/188921611-59302748-c098-485f-a2d5-e429f224075e.png)

Step 2- In the search bar, type *Azure Synapse Analytics* and hit *Create* at the bottom of the returned result.
![image](https://user-images.githubusercontent.com/95705084/188921819-a4bee48b-c77c-48d9-9614-07101650b787.png)

Step 3- Once you have selected *Create*. a screen redirect will take place and you will be prompted to input the following information:

  - **Subscription**: Ensure the proper subscription is set correctly.
  - **Resource Group**: Ensure the proper resource group is set, or select "*Create New*" directly below the box if a new group is needed.
  - **Managed Resource Group**: This is optional and can be used if you are looking to segregate all resources for Synapse within your respective resource group.
  - **Workspace Name**: This will be the name of the Synapse Workspace. Please ensure that you select a name you are comfortable with as the process for making an update can take time.
  - **Use Spark on Cosmos**: This is optional and only required if the intent is to setup an analytical store between Cosmos and Azure Synapse. More information on this can be found here - Azure Synapse Link for Azure Cosmos DB, benefits, and when to use it | Microsoft Docs![image](https://user-images.githubusercontent.com/95705084/188923602-65a4ec50-0bf5-4e77-9c81-93e345a9790a.png)
  - **Region**: Select the region in which the resource(s) are to be deployed. 
  - **Select Data Lake Storage Gen2**: Select the option that is applicable-
          - If an existing ADLS Gen2 is already stood up and you would like it to be used, select if from the drop down.
          - If a new ADLS Gen2 needs to be configured, please select the "*Create New*" hyperlink below the drop down box. Please provide the resource with an account name and a file system name. (NOTE: File System name is a folder created within the ROOT of ADLS Gen2)
          - Please take note of the ADLS resource as this will need to be revisted when configuring permissions throughout the Synapse workspace.
  - After this has been filled out, select the security tab at the bottom of the screen for next steps.
![image](https://user-images.githubusercontent.com/95705084/188924895-79797fe7-226b-42ec-88de-8b3ebfa49f96.png)



