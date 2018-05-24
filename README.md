Methods to Migrate SQL Server 20xx to Azure SQL Database
In this lab, you learn about the primary methods for migrating a SQL Server 2005 or later database to a single or pooled database in Azure SQL Database.
There are several sections to this exercise, which should take about 60 minutes to complete all sections.
	• Lab Setup: Preparing a VM with SQL Server 2017 and deploying an empty SQL Database, both in Azure Government. Restoring a database from a backup.
	• BACPAC Migration: Export a database from SQL Server 2017 to BACPAC format, export using SQL Server Management Studio
	• DMA Migration: Using the Data Migration Assistant to test compatibility, export to Azure SQL Database
	• Transactional Replication Migration: Not covered in this lab 
	
This lab will begin with preparing our lab environments for restores, migrations, and testing. 


Lab Setup

 
	Create the Lab Environment in Azure Government
	1. Log in to https://portal.azure.us
	2. Click “Create a resource,” and search for:
 “Free SQL Server License: SQL Server 2017 Developer on Windows Server 2016”
	3. Select the image and click “Create”
	4. Provide the following information, unique to your machine:

		Press “OK” to validate and proceed 
	5. Select a moderately sized machine to host the SQL Server instance, such as a DS2_v2 VM

	6. Select all of the defaults within the optional “Settings” pane:

		Press “OK” to validate and proceed.
		 
	7. On the “SQL Server Settings” pane, select the defaults and click “OK” to proceed:

		 
	8. On the “Summary” page, press “OK” to deploy and proceed:

	 
	Prepare SQL Database
	In the Marketplace, search for “SQL Database” and click “Create.”
	9. Fill in all information marked with an asterisk:

	10. Click the “Pricing Tier” tab and select the "S3 - 100 DTUs" option, 400 GB of storage, and hit “Apply”:

	11. Press the “Create” button to begin the provisioning process
	Once the machine and database are provisioned, proceed to the next section.
	
	
	Azure VM Database Preparation
	
	1. Navigate through the portal blades to your resource group for your lab and locate your virtual machine (Sort by Type, if needed), then click on it.
	

	
	2. Connect to the machine using the RDP option.

	
	Press "Connect," download the RDP file, open it, type in your credentials for the VM (e.g. Instructor, password) and click Connect.
	
	3. After logging in, Download the following file:
	https://sqlmigrationstorage.blob.core.usgovcloudapi.net/labfiles/AdventureWorksLT.bak
	(Note: You may need to disable IE Enhanced Security Configuration for this task)

	
	Move the file to this folder on your server:C:\Program Files\Microsoft SQL Server\MSSQL14.MSSQLSERVER\MSSQL\Backup
	
	4. Next, open SQL Server Management Studio from the Start Menu 

	
	Then connect to the database instance:

	
	Additionally, connect to your SQL Database as well, this is the name you provided for Step 9 of the lab preparation. Simply append ".database.usgovcloudapi.net" to the end of the server name:


	5. Next, restore the database to your local server by opening the server instance, right-clicking on "Databases" and selecting "Restore Database":

	
	6. Select the AdventureWorksLT backup file you previously copied to the "Backups" folder and press "OK":

	
	This completes the lab preparation sequence. You are ready to begin test migrations
	



Explaining Migration Planning to a single database or a pooled database
There are two primary methods for migrating a SQL Server 2005 or later database to a single or pooled database in Azure SQL Database. The first method is simpler but requires some, possibly substantial, downtime during the migration. The second method is more complex, but substantially eliminates downtime during the migration.
In both cases, you need to ensure that the source database is compatible with Azure SQL Database using the Data Migration Assistant (DMA). SQL Database V12 is approaching feature parity with SQL Server, other than issues related to server-level and cross-database operations. Databases and applications that rely on partially supported or unsupported functions need some re-engineering to fix these incompatibilities before the SQL Server database can be migrated.

Note
To migrate a non-SQL Server database, including Microsoft Access, Sybase, MySQL Oracle, and DB2 to Azure SQL Database, see SQL Server Migration Assistant.
Method 1: Migration with downtime during the migration
Use this method to migrate to a single or a pooled database if you can afford some downtime or you are performing a test migration of a production database for later migration. For a tutorial, see Migrate a SQL Server database.
The following list contains the general workflow for a SQL Server database migration of a single or a pooled database using this method. 


	1. Assess the database for compatibility by using the latest version of the Data Migration Assistant (DMA).
	2. Prepare any necessary fixes as Transact-SQL scripts.
	3. Make a transactionally consistent copy of the source database being migrated or halt new transactions from occurring in the source database while migration is occurring. Methods to accomplish this latter option include disabling client connectivity or creating a database snapshot. After migration, you may be able to use transactional replication to update the migrated databases with changes that occur after the cutoff point for the migration. See Migrate using Transactional Migration. 
	4. Deploy the Transact-SQL scripts to apply the fixes to the database copy.
	5. Migrate the database copy to a new Azure SQL Database by using the Data Migration Assistant.
	
Note
Rather than using DMA, you can also use a BACPAC file. See Import a BACPAC file to a new Azure SQL Database.We will also cover this in a subsequent lab section

Optimizing data transfer performance during migration
The following list contains recommendations for best performance during the import process.
	• Choose the highest service level and performance tier that your budget allows to maximize the transfer performance. You can scale down after the migration completes to save money. 
	• Minimize the distance between your BACPAC file and the destination data center.
	• Disable auto-statistics during migration
	• Partition tables and indexes
	• Drop indexed views, and recreate them once finished
	• Remove rarely queried historical data to another database and migrate this historical data to a separate Azure SQL database. You can then query this historical data using elastic queries.
Optimize performance after the migration completes
Update statistics with full scan after the migration is completed.






USING DMA:

	1. Still connected to the virtual machine, download and install the Data Migration Assistant: https://www.microsoft.com/download/details.aspx?id=53595
	Install using all defaults
	After completion, go to Start --> Data Migration Assistant
	
	2. After the DMA opens, you will see a screen similar to the following:

	
	3. Click "New" and provide some basic details of the Project you are working on - but we will simply perform an assessment only at this time:

	
	4. Provide the default checks and proceed:

	
	5. Select the local server on which you are running to locate the AdventureWorksLT database - 


	6. Select the AdventureWorksLT database as the source:

	
	7. Start the Assessment:
	

	
	8. And finally, export the report:

	
	After saving the report, open it and view the material it provides to understand how the tool operates.
	
	9. Next, Create a new project, this time selecting "Migration," instead, filling in the proper types and scope:
	

	
	10. Select the source server, the database name, and check both boxes before pressing "Connect":

	
	11. Connect to your SQL Database instance, which is named "<DatabaseName>.database.usgovcloudapi.net":
	Note: The previous example provided sqlmigrationlab.database.usgovcloudapi.net

	
	12. Proceed with all objects selected and click "Generate SQL Script":

	
	13. You may optionally "Save" the generated script, or simply "Deploy Schema" and subsequently "Migrate Data" buttons, in succession:


	14. On the "Select tables" tab, click "Start data migration": 

	
	15. The migration should proceed -- validate there were no warnings or errors:

	
	16. Last, open SQL Server Management Studio and connect to the database on the Azure SQL Database server to validate the database was successfully migrated:
	Validate: 






EXPORT TO BACPAC:
A data-tier application (DAC) is a logical database management entity that defines all of the SQL Server objects - like tables, views, and instance objects, including logins – associated with a user’s database. A DAC is a self-contained unit of SQL Server database deployment that enables data-tier developers and database administrators to package SQL Server objects into a portable artifact called a DAC package, also known as a DACPAC. 
A BACPAC is a related artifact that encapsulates the database schema as well as the data stored in the database. 

Read more information on the subject here: https://docs.microsoft.com/en-us/sql/relational-databases/data-tier-applications/data-tier-applications?view=sql-server-2017

	1. To begin this process, connect to the SQL Server using SQL Server Management Studio and right-click the "AdventureWorksLT database --> Tasks --> Export Data-tier Application…"
	Note: If you Azure SQL Database AdventureWorksLT still exists, delete it now using SSMS or the Azure Portal.

	
	2. Select a location to save the file, preferable the "Backup" directory used previously: 

	
	3. Finish and view the Summary:
	

	
	
	If you have issues with this process, you can copy and use this file: 
	https://sqlmigrationstorage.blob.core.usgovcloudapi.net/labfiles/AdventureWorks.bacpac
	
	4. Once your BACPAC export is complete, next use SSMS to connect to the SQL Database by right-clicking "Databases" and then selecting "Import Data-tier Application": 
	

	
	5. Proceed through the wizard:

	
	
	6. Import the file you previously backed up (BACPAC):
	

	
	7. Provide a new database name, such as "AdventureWorksLT", and select the "S3" Service objective:

	
	
	8. Finalize the import:

	
	9. Ensure there are no errors:

	
	10. Connect to the database and browse the schema and data using SSMS:



	13. Using SSMS, downsize the database using the following T-SQL Command:
	ALTER DATABASE AdventureWorksLT MODIFY ( EDITION = 'Standard' , MAXSIZE = 250 GB , SERVICE_OBJECTIVE = 'S1' ); 


Advanced Functionality of SQL Database:
On your own, without guidance, attempt to perform the following configurations:
	1) Configure a firewall setting using the portal, optionally, through T-SQL
	2) Configure Long-Term backup retention
	3) Review and configure Auditing & Threat Detection settings
	4) Set your Active Directory Admin
	5) Review the metrics of the database activity
	6) Configure Geo-Replication
	7) Create Tags for your server
	8) Move your database into an elastic pool
	9) Use the Azure CLI to create a database
	10) Use PowerShell to create a database
	11) Learn how to connect to Azure Government Resources using Visual Studio
	12) Review SQL Database to SQL Server feature parity: https://docs.microsoft.com/en-us/azure/sql-database/sql-database-features


