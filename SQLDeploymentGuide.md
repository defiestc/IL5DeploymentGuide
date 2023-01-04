# Azure SQL Overview

## Table of Contents

[Prerequisites](#prerequisites)

[Helpful Links](#helpful-links)

[Azure SQL Considerations](#azure-sql-considerations)

[SQL on Azure VMs](#sql-on-azure-vms)

[Private Links for Azure SQL Database](#private-links-for-azure-sql-database)

[Deployment](#deployment)

[Deploy SQL Azure VMs](#deploy-sql-azure-vm)

[Deploy Azure SQL database](#deploy-azure-sql-database)

[Encryption](#encryption)

[Key Vault Integration for SQL VMs](#key-vault-integration-with-azure-sql-vms)

[Customer Managed keys for SQL Availability Groups](#customer-managed-keys-for-sql-availability-groups)


## Helpful Links

The following links can be used as references for additional information within the deployment guide, if needed. This is not an exhaustive list as Azure documentation is updated frequently.

[Register with SQL IaaS Extension (Windows) - SQL Server on Azure VMs](https://learn.microsoft.com/en-us/azure/azure-sql/virtual-machines/windows/sql-agent-extension-manually-register-single-vm?view=azuresql&tabs=bash%2Cazure-cli)

[Register multiple SQL VMs in Azure with the SQL IaaS Agent extension - SQL Server on Azure VMs](https://learn.microsoft.com/en-us/azure/azure-sql/virtual-machines/windows/sql-agent-extension-manually-register-vms-bulk?view=azuresql)

[Quickstart: Create a private endpoint by using the Azure portal - Azure Private Link](https://learn.microsoft.com/en-us/azure/private-link/create-private-endpoint-portal?tabs=dynamic-ip)

[Create SQL Server on a Windows virtual machine in the Azure portal - SQL Server on Azure VMs](https://learn.microsoft.com/en-us/azure/azure-sql/virtual-machines/windows/sql-vm-create-portal-quickstart?view=azuresql&tabs=conventional-vm#configure)

[Create a single database - Azure SQL Database](https://learn.microsoft.com/en-us/azure/azure-sql/database/single-database-create-quickstart?view=azuresql&tabs=azure-portal)

[Integrate Key Vault with SQL Server on Windows VMs in Azure (Resource Manager) - SQL Server on Azure VMs](https://learn.microsoft.com/en-us/azure/azure-sql/virtual-machines/windows/azure-key-vault-integration-configure?view=azuresql)

[Set up Transparent Data Encryption (TDE) Extensible Key Management with Azure Key Vault - SQL Server](https://docs.microsoft.com/en-us/sql/relational-databases/security/encryption/setup-steps-for-extensible-key-management-using-the-azure-key-vault?view=sql-server-ver15&tabs=portal#step-1-set-up-an-azure-ad-service-principal)

## Prerequisites
### Azure SQL Considerations

When trying to decide between Azure SQL options the following factors should be considered by the customer: 
1. Current operational processes:
	* Processes that need to be carried over to Azure:
		* Example:  Data regularly needs to be moved from low side to high side.  
			* How is this being done today?  
			* Can this process become more inconvenient in Azure?  
	* Processes that can be improved:
		* Example:  Monitoring 
			* Currently reporting on number of SQL instances being up, database status, storage thresholds, memory contention, etc.
			* No need to report on the infrastructure being up in Azure SQL PaaS
	* Would cross-subscription resource moves be required? 
		* Examples:  data refresh from prod subscription to development subscription 
			* Some resources are not supported for cross subscription moves
2. Authentication: what kind of authentication do the applications targeted for Azure support? 
	* Azure SQL PaaS options currently supports SQL auth, which is usually a STIG violation (a waiver can be requested)
	* Azure AD authentication which would require the customers to federate their on-prem users to Azure AD
	* Managed Identities
3. Storage: their database storage requirements may determine the service tier of the resources
4. Disaster recovery requirements:  locally redundant vs zone redundant 
5. Security:
	* Network security
	* Database security (TDE):  Customer Managed Key vs. Service Managed Key
6. Migration strategy:
	* Consideration:  How much downtime can the app(s) tolerate?
    * Options:  Backup/Restore, bacpac, Database Migration Service, Azure data studio, Database migration assistant   

### SQL on Azure VMs

SQL Server VM Image

In most cases, customers deploying SQL server on Azure VMs will have their own image instead of using a marketplace image.  In that case, the customers must register the SQL Server VM with the SQL IaaS Agent extension to unlock managed features such as automated backups and patching.  Here are pre-requisites for the IaaS extension:
* The latest version of Azure CLI or Azure PowerShell (5.0 minimum).
* A minimum of .NET Framework 4.5.1 or later.
* Register the subscription with the Microsoft.SqlVirtualMachine resource provider:
	* Open the Azure portal and go to All Services.
	* Go to Subscriptions and select the subscription of interest.
	* On the Subscriptions page, select Resource providers under Settings.
	* Enter sql in the filter to bring up the SQL-related resource providers.
	* Select Register for the Microsoft.SqlVirtualMachine provider.
	

### Private Links for Azure SQL Database

Create a Private Endpoint

#### Prerequisites: Resource Group, Azure SQL Server, Virtual Network

1. In the search box at the top of the portal, type "Private endpoint". Select Private endpoints from the listed Services.
2. Select + Create in Private endpoints.
3. In the Basics tab of Create a private endpoint, enter the following information:
	* Subscription
	* Resource group
	* Name of the private endpoint
	* Region
4. In the Resource tab, enter the following information:
	* Connection method
	* Subscription
	* Resource:  the name of the Azure SQL DB server
	* Target sub-resource:  "sqlServer"
5. In the Networking tab, enter the following information:
	* Virtual network
	* Subnet
	* Private IP configuration
	* Private DNS integration
6. In the Review and Create tab, click Create.

Approval Process
1. Navigate to the Azure SQL database server selected in step 4c in the previous section.
2. Click on "Show networking settings" on the overview blade , in "Essentials" section at the top of the page.
3. Click on Private access
4. Verify that the private endpoint (created in the previous section) has the Connection state of "Approved"


## Deployment
### Deploy SQL Azure VM

Azure SQL Server VM using a custom image

There are 6 basic steps to creating a custom SQL Server VM image for use in Azure.
1. Provision a new base Windows Server VM
2. Download the SQL Server installation media
3. Run SQL Server setup to prepare an image
4. Configure Windows to complete the installation of SQL Server
5. Capture the image and add it to the Azure VM image gallery
6. Create a new VM instance using the custom SQL Server image

Azure SQL Server VM using a marketplace image

1. Sign in to the Azure portal using your account.
2. In the search bar at the top of the portal, type "Azure SQL".  Select Azure SQL from the listed Services.
3. Select +Add to open the Select SQL deployment option page. You can view additional information by selecting Show details on the SQL virtual machines tile.
4. For conventional SQL Server VMs, select one of the versions labeled Free SQL Server License... from the drop-down. 
5. Select Create
6. On the Basics tab, provide the following information:
	* In the Project Details section, select your Azure subscription and then select Create new to create a new resource group. 
7. Under Instance details:
	* Type a name for the Virtual machine name.
	* Choose a location for your Region.
8. Choose an Availability option (Availability sets/ Availability zones)
9. In the Image list, select the image with the version of SQL Server and operating system you want.
10. Choose to Change size for the Size of the virtual machine.
11. Under Administrator account, provide a username, and a password. 
12. Under Inbound port rules, choose Allow selected ports and then select RDP (3389) from the drop-down.
13. On the SQL Server settings tab, configure the following options:
	* Under Security & Networking, select "Private (within Virtual Network) and change the port number to avoid using a well-known port number
	* Under SQL Authentication, Choose "Disable"
	* Provide Key vault information as described in section: Key Vault Integration for SQL VMs
	* Change any other settings if needed, and then select Review + create.
14. On the Review + create tab, review the summary, and select Create to create SQL Server, resource group, and resources specified for this VM.


### Deploy Azure SQL Database

Create a single database in the Azure portal:

1. In the search bar at the top of the Azure Portal type "Sql databases", and select SQL databases from the listed Services.
2. Select + Create, or use the Create SQL Database button.
3. On the Basics tab of the Create SQL Database form, under Project details, select the desired Azure Subscription.
	* For Resource group, select Create new or pick an existing one from the drop down list
	* Enter a database name
	* For Server, select Create new, and fill out the New server form with the following values:
		* Server name
		* Location: Select a location from the dropdown list.
		* Authentication method
			* Server admin login and Password if SQL authentication is chosen
	* Leave Want to use SQL elastic pool set to No.
	* Under Compute + storage, select Configure database and choose your service tier and compute tier
	* Under Backup storage redundancy, choose a redundancy option for the storage account where your backups will be saved. 
4. Select Next: Networking at the bottom of the page.
	* For Firewall rules, set Add current client IP address to Yes. Leave Allow Azure services and resources to access this server set to No.
	* On the Networking tab, for Connectivity method, select Private endpoint. Configure a new private endpoint as explained in section: Private Endpoint for Azure SQL Database. If a private endpoint is already configured for the VNET, it will appear here and the next two steps will appear grayed out.
	* Under Connection policy, choose the Default connection policy, and leave the Minimum TLS version at the default of TLS 1.2.
	* On the Security page, you can choose to start a free trial of Microsoft Defender for SQL, as well as configure Ledger, Managed identities and Transparent data encryption (TDE) if you desire. Select Next: Additional settings at the bottom of the page.
5. On the Additional settings tab, in the Data source section, for Use existing data, select "none".  You can also configure database collation and a maintenance window.
6. Select Review + create at the bottom of the page.
7. On the Review + create page, after reviewing, select Create.


## Encryption
### Key Vault integration with Azure SQL VMs

1. To create a single database in the Azure portal, type "Sql database" in the search bar at the top in azure portal, and click "Create"
2. On the Basics tab of the Create SQL Database form, under Project details, select the desired Azure Subscription.
	* For Resource group, select Create new or pick an existing one from the drop down list
	* Enter a database name
	* For Server, select Create new, and fill out the New server form with the following values:
		* Server name
		* Location: Select a location from the dropdown list.
		* Authentication method
			* Server admin login and Password if SQL authentication is chosen
	* Leave Want to use SQL elastic pool set to No.
	* Under Compute + storage, select Configure database and choose your service tier and compute tier
	* Under Backup storage redundancy, choose a redundancy option for the storage account where your backups will be saved. 
3. Select Next: Networking at the bottom of the page.
	* For Firewall rules, set Add current client IP address to Yes. Leave Allow Azure services and resources to access this server set to No.
	* On the Networking tab, for Connectivity method, select Private endpoint. Configure a new private endpoint as explained in section: Private Endpoint for Azure SQL Database
	* Under Connection policy, choose the Default connection policy, and leave the Minimum TLS version at the default of TLS 1.2.
	* On the Security page, you can choose to start a free trial of Microsoft Defender for SQL, as well as configure Ledger, Managed identities and Transparent data encryption (TDE) if you desire. Select Next: Additional settings at the bottom of the page.
4. On the Additional settings tab, in the Data source section, for Use existing data, select "none".  You can also configure database collation and a maintenance window.
5. Select Review + create at the bottom of the page.
6. On the Review + create page, after reviewing, select Create.

### Customer Managed keys for SQL Availability Groups

TDE Encryption with Customer-Managed Keys
1. Register an App in App registration in the portal to get the Object ID and Secret (see previous section)
	* Create a Secret, note down the secret value. Note down the Application (client) ID found on the overview page. 
2. Create a Key Vault 
3. Import (or create) a key in the key vault. This key will be used to encrypt databases with TDE. 
4. Set Access policy in the Key Vault – Use Key Mgmt template and add Unwrap Key , Wrap Key
	* Set access policy for SQL Server managed ID 
	* Set access policy for Application registration (from step 1) managed ID 
5. Integrate SQL Server with AKV 
	* Go to SQL Server in the portal, click on Azure Key Vault integration and enable Key Vault
	* You will need the Key Vault URL, the application ID of the app registration and the secret value of the secret from step 1. 
	* Give the credential a name. The credential created here needs to be mapped to the SQL service account to create an Asymmetric Key.
6. Install SQL Connector for AKV, if required. Marketplace image of Enterprise SQL Server 2019 on Windows Server 2019 already comes with it. 
7. Create a registry key and give full permissions to SQL Service account
	* Create a new registry key in HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\
	* Name it “SQL Server Cryptographic Provider” (without the quotes)
	* Give full permission to the SQL Server service account on this key.
8. Steps 4 (Set acccess policy for SQL Server managed ID), and 5-7 should be repeated on all servers in Always On AGs
9. Configure SQL Server
	* Map the credential created in step 5 to the service account running SQL Service. Repeat on all SQL instances in Always On AGs.

			ALTER LOGIN [SQLserver4TDEDe\sqladmin]
			ADD CREDENTIAL [AKVCred4SQLServiceAcct];  --this cred was created in step 5
		 
	* Create an Asymmetric Key by opening the Key created in the Key Vault.  Repeat on all SQL instances in Always On AGs.

			CREATE ASYMMETRIC KEY AsymKeyTDE
			FROM PROVIDER  [AzureKeyVault_EKM_Prov] 
			WITH PROVIDER_KEY_NAME = 'AsymSQLTDE',  --This key was created in step 3
			CREATION_DISPOSITION = OPEN_EXISTING;


	* Create a Login that will associate the asymmetric key to this login. This login needs a Cred. Repeat on all SQL instances in Always On AGs.
			
			CREATE LOGIN Login4_AsymKeyTDE
			FROM ASYMMETRIC KEY AsymKeyTDE; --this key was created in step the second sub-step in step 9
		 
	* Drop Cred from service account. Repeat on all SQL instances in Always On AGs.
			
			ALTER LOGIN [SQLserver4TDEDe\sqladmin]
			drop CREDENTIAL [AKVCred4SQLServiceAcct];  
		 
	* Add the credential mapping to the new Login. Repeat on all SQL instances in Always On AGs.
			
			ALTER LOGIN Login4_AsymKeyTDE
			add CREDENTIAL [AKVCred4SQLServiceAcct];
		 
	* Create a test database (if a database doesn’t exist) that will be encrypted with the Azure key vault key
			
			CREATE DATABASE TestTDE
		 
	* Create an ENCRYPTION KEY using the ASYMMETRIC KEY. Only do this on the PRIMARY server (of each group for the database/s required)
			
			Use TestTDE
			GO
			CREATE DATABASE ENCRYPTION KEY
			WITH ALGORITHM = AES_256
			ENCRYPTION BY SERVER ASYMMETRIC KEY AsymKeyTDE; --key created in the second sub-step in step 8 
		 
	* Enable TDE by setting ENCRYPTION ON. Only do this on the PRIMARY server (of each group for the database/s required)
			
			ALTER DATABASE TestTDE
			SET ENCRYPTION ON;
		 
	* Check database encryption status
			
			SELECT
			db.name,
			dm.encryption_state_desc,
			db.is_encrypted,
			dm.encryption_state,
			dm.percent_complete,
			dm.key_algorithm,
			dm.key_length
			FROM
			sys.databases db
			LEFT OUTER JOIN sys.dm_database_encryption_keys dm
			ON db.database_id = dm.database_id;
			GO
	 
10.  Restore a TDE encrypted database to another SQL Instance:
	* Repeat the first 5 sub-steps in step 9
	* Restore database
	* Check status using the last sub step in step 9
 

