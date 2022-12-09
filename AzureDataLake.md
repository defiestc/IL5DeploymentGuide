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
Step 5. **Important Step** If this is not checked off, this service will be rendered obsolete and will not be an ADLS Gen2 service. 
          
