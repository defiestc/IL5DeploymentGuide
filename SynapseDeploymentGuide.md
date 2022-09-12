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

Step 4 - For Authentication method, ideally we would only select **Use Only Azure Active Directory (Azure AD authentication)** however, during initial configuration, it is safe to select "*Use both local and Azure Active Directory authentication*" for testing purposes and to ensure everything is setup correctly. We will disable local admin access once everything is validated. More information on this can be found here -  Authorize server and database access using logins and user accounts - Azure SQL Database & SQL Managed Instance & Azure Synapse Analytics | Microsoft Docs![image](https://user-images.githubusercontent.com/95705084/188926251-55225846-3e45-47ea-b1f2-aed05456a47f.png)

![image](https://user-images.githubusercontent.com/95705084/188926284-e1296326-58da-452a-9a5e-8add2d10b6d7.png)

Step 5 - Select the check box **Allow network access to Data Lake Storage Gen2 account** This will assign the mananaged identity within Synapse to the Azure Data Lake store and allow for secure access between the two accounts. More information and steps will be provided on how to validate and test in upcoming sections. (NOTE: If a new ADLS account was provisioned, this will not allow you to select this.)

Step 6 - Select the "*Enable*" option for workspace encryption. This will allow for encryption of data at rest via customer mananaged keys that will be stored and secured within Azure Key Vault. 
      - Ensure the proper key vault resource and keys are selected when configuring this option. (NOTE: This should have been done prior to creating Synapse Workspace)
      - Ensure that System assigned is selected and not user assigned. We want to rely solely on the system versus the uesr in the event an account was compromised or disabled\deleted. Additional permissions around the system mananaged identity will need to be granted and steps on how to do so will be provided throughout the later stages of this guide.
![image](https://user-images.githubusercontent.com/95705084/188928145-1f447c41-102c-48ba-84c7-7ab8c583243e.png)

Step 7 - Once these options have bene selected and configured, please click the *"Next: NEtworking*" tab at the bottom to be redirected to the networking portion of Synapse.
In this screen, you will note only a few settings, however each setting plays a huge factor in how Synapse needs to be secured and accessed. As the progression of this guide takes place, more and more options to secure endpoints across Azure services to Synapse will need to be done. This is a primary step which enables additional steps to take place. (**NOTE**: Please keep in mind that once these Network settings have been configured, they cannot be changed and a new workspace would have to be created. Please ensure that the pre-requisites outlined above have been identified and are ready for deployment)
    - Step 7a. Please ensure that "Enable" is selected for Managed virtual Network. The logic behind this selection is to ensure that Synapse is managing the Virtual network traffic and rules versus it being done at the Vnet level. 
When enabling Managed Virtual Network, a new series of options will pop-up, please select the following as below: 
    - Step 7b. Select "Yes" on "Create Mananaged Private Endpoint to Primary Data storage account". This will create a private and direct line of communication, or "secure traffic" to the Azure Data Lake Gen2 account.
    - Step 7c. This step is optional and can be done at a later time but if all endpoints have already been identified in which Synapse needs to communicate with securely, they can be added as shown below.
    - Step 7d. Ensure that "Public Network access to workspace endpoints" is disabled. This is important as we want to disallow public network access and keep it as secure as possible.
![image](https://user-images.githubusercontent.com/95705084/189729058-c061a106-1ff7-417d-9ea8-05db1b152ab3.png)

Step 8 - After this has been set, please hit "Review + Create". (**NOTE**: If resource tagging is being done, please select "Tags" and tag the resource based on what policy is in place.
![image](https://user-images.githubusercontent.com/95705084/189729357-aad0bf4f-538f-4f1b-beab-9ddc8cc18a85.png)

Step 9 - After the resource has been created, select it and open Synapse Studio --> Click on the Toolbox Icon to the bottom left labeled "Manage". Once inside of the Manage screen, please click into the Security portion and select "Managed Private Endpoints". This will show 3 private endpoints that were created upon the creation of Synapse itself, with one endpoint needing approval before being able to successfully leverage it.
![image](https://user-images.githubusercontent.com/95705084/189730082-5b9e3ec0-8822-4c34-9eda-539a68f41baf.png)

Step 9a - Synapse requests approval of the Azure Data Lake resource to ensure that it is the correct resource and the proper rules are set in place. In order to approve ADLS Gen2, please click into the resource --> Manage approvals in Azure Portal.
![image](https://user-images.githubusercontent.com/95705084/189731448-d6996b9b-4fb6-4a7f-aadf-5618cbf3cb26.png)

Step 9b - After clicking into it, a redirect will take place in which it will ask to approve or reject the endpoint. Please select the check box --> Select Approve
![image](https://user-images.githubusercontent.com/95705084/189731727-dea80472-7fe2-4092-9e3c-8843a6a769af.png)

Step 9c - Once approved, the connection state will change from Pending to Approved and the resource can now be used. (**NOTE**: Please take note that the status within Synapse for the resource will tak ea few minutes to update. Continue to refresh if needed until the approval state is showing "Approved")
![image](https://user-images.githubusercontent.com/95705084/189732593-833be861-db0c-483b-a37b-1583826ccdf1.png)

### Creating Private Endpoints

As data sources are identified and additional resources need to communicate with Synapse, additional private endpoints need to be created to ensure that all traffic to and from Synapse are secure in nature. The next steps provided below are interchangeable to any endpoint being leveraged but for this example, Azure Cosmos DB is being called. 

Step 10 - 





