## Azure Data Lake Service Gen 2 

### This guide was put together with the intent of walking through a deployment of the Azure Data Lake Gen 2 service and how to go about securing it ensuring it falls within IL5 compliancy. 

### Deployment Guide for Azure Data Lake Service Gen2

Step 1. Within the Azure Portal-->Create a Resource, perform a search for Storage Account and select **Create**.

Step 2. Fill in the following information:
   - Subscription: Name of subscription this will be deployed in.
          
   - Resource Group: Name of Resource Group this will be deployed in.
                    
   - Storage Account Name: Provide a name for the ADLS Gen2 service.
                    
   - Region: Select a region this resource will be deployed to.
                    
   - Performance: Two options can be selected-
                    
        - Standard: Recommended for general hot, warm, cold storage scenarios and general file storage.
                              
        - Premimum: Recommended for scenarios that require low latency such as transactional processes against the Gen2 service.
                              
 - Redundancy: Based on SLA and need, select the approriate protection layer for this service.
                    
![image](https://user-images.githubusercontent.com/95705084/206785300-fc403311-7c79-4364-addc-5d1d009fb8c2.png)

Step 3. Select **Next** at the bottom of the screen to be directed to the **Advanced** configuration of the setup.

Step 4. Within the **Security** portion of the configuration, ensure the following is checked off:

   - Require secure transfer REST API operations.
   
   - Enable storage access key access.
   
   - Minimum TLS version is 1.2

Step 5. **Important Step** If this is not checked off or followed, this service will be rendered obsolete and will not be an ADLS Gen2 service.

   - The Data Lake Storage heirarchical namespace accelerates big data analytics workloads and enables file-level access control lists (ACLs).
   
        - Enable hierarchical namespace - **Please check this box** When selected a series of features will become available. Please select only if needed.
        - Enable SFTP - Optional
        - Access Tier - Select as needed. Please ensure the proper tier is selected.
Step 6. Select **Next** at the bottom of the screen to be directed into the Networking portion of the setup. Ensure that **Disable public access and use private access** is selected.

Step 7. For routing preference, ensure that **Microsoft Network routing** is selected. This ensures your routing is traversing across Microsoft's fabric and is protected versus traversing across the internet. Select **Next** once completed to be directed to the Data Protection portion of the configuration.

Step 8. The Recovery portion of this configuration can be left as is unless a specific policy is defined that needs to be followed. Please adjust as needed. Select **Next** at the bottom of the screen to be directed to the Encrpytion portion of the configuration.

Step 9. For encryption type, please ensure Customer-Managed Keys are selected. This will ensure that the ADLS Gen2 resource will be protected by the approriately managed keys within Azure Key Vault. Azure Key Vault is a pre-requsite in this guide and is required - if this has not been setup, please follow the instructions within the Deployment Guide for deploying a Key Vault service.

   - Enable support for customer-managed keys: Please ensure All service types (blobs, files, tables, and queues) is selected.
   - Encrpytion key: Select a key vault and key would leverage the Azure Key Vault service. 
   - Enable infrastructure encryption: Ensure this is selected for an additional layer of encryption.
 
Step 10. Select **Next** at the bottom of the screen until it arrives at the **Review** section of the configuration. Select **Review and Create** and the service will be created.
