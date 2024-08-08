Azure Data Engineering - Real Time End to End Project w/ AdventureWorks Database

Setup:

1: Project Overview
2: On-Prem & Cloud Setup
3: Data Extraction

ETL:-

4: Data Transformation
5: Data Loading
6: Data Reporting

7: Pipelines Testing/Automation

------------------------------------------------------------------------------------------------------

Part 1: Project Overview:

Source -> Ingestion -> Preparation (Layer 1 & 2) -> Transformation -> Loading -> Reporting
SQL Server -> Azure Data Factory -> ADLS Gen2 -> Azure Databricks -> Azure Synapse Analytics -> PowerBI

------------------------------------------------------------------------------------------------------

Part 2: OnPrem and Cloud Setup:

OnPrem Setup:-

- Download AdventureWorks database: https://learn.microsoft.com/en-us/sql/samples/adventureworks-install-configure?view=sql-server-ver16&tabs=ssms

- Select adventure-works-2008r2-lt.bak

- Open MSL SQL Server -> Databases -> Restore Databases -> Select downloaded DB

- Create LOGIN demo with password = "ponytail538"
- Create user demo for login demo
- Assign DB read access to created user: Go to Logins -> demo -> Properties -> DB Role membership -> db_datareader


Cloud Setup:

- Launch Azure Portal
- Navigate to respective project
- Go to Azure Key Vault -> Generate/Import -> Upload Options: Manual, Name: username, Secret Value: demo
- Again -> Generate/Import -> Upload Options: Manual, Name: password, Secret Value: ponytail538

Now login credentials have been saved to the Azure Cloud

------------------------------------------------------------------------------------------------------

Part 3: Data Ingestion: Building the data injection pipeline

- Go to Azure Data Factory
- Establish connection between the Azure Data Factory and On Prem SQL server
- Go to Data Factory -> Manage - Integration Runtimes
- Select new self hosted Azure runtime, name it selfHostedIR
- Click Option 1: Launch Express Setup on this computer
- Launch Microsoft Integration Runtime on the PC: Check for connected to cloud service with green tick at bottom
- ADF -> Author -> + Sign -> Pipeline -> Pipeline -> Name it picopytables
- Activities -> Lookup -> Drag and Drop to Workspace (Get get tables from source)
- Use Script from demo
- Name lookup table as Get_all_tables_lookup
- Go to settings and select source dataset -> Click New -> Select SQL Server -> Name it onpremdbtables
- Create linked service -> name it sqlserverlinkedservice -> Connect via integration runtime -> SelfhotedIR
- Copy paste server name from SSMS -> Copy DB name from SSMS
- Authentication Type -> SQL Authentication -> Test Connection -> Username = sa (?) -> Password = ? -> Click create
- Use the same SQL query to check if same resultant tables are achieved
- Search for ForEach iterator (to copy all tables) in activities, drag it to canvas and link it up with Lookup activity
- In Activity Outputs, write @activity('Lookuptables').output.value
- Click on activites -> go to pencil icon -> you will reach a blank canvas
- Search for and drag Copy Data activity to the canvas area
- Rename it to CopyEachtableForEach
- Go to source and select new source dataset -> select SQL server -> In linked service choose LinkedSQLServer and Leave table name blank
- In Use Query select Query -> click add dynamic content and use following query:
@{concat('SELECT * FROM',item().SchemaName,'.',item().TableName)}
- Go to sink and select new or + sign -> select Azure Data Lake Gen2 (for data store) -> select Parquet format (easier to retrieve data column based data structure unlike row based ones)
- Name it ParquetDatasetLayer1, create new Linked Service -> rename it ADFLinkedLayer1 -> Connect via AutoResolveIntegrationRuntime -> Authentication Type: Account Key -> Select relevant subscription -> Select storage account name eg: adlsgen2demo -> Test Connection -> Click create
- Under File Path: select browse and click Layer-1 -> click OK
- To create new folder structure for sink dataset, go to ParquetDatasetLayer1 click parameters, click new, create parameters schemaname and tablename
- Go to sink -> schemaname -> add dynamic content -> click ForEach copytables and append .schemaname so that Pipeline expression builder canvas looks like: @item().SchemaName
- Go to sink -> schemaname -> add dynamic content -> click ForEach copytables and append .tablename so that Pipeline expression builder canvas looks like: @item().TableName
- Open sink dataset -> Open -> Connection Tab -> Go to directory structure and add dynamic content:
@{concat(dataset().schemname, '/',dataset().tablename)}
- Open sink dataset -> Open -> Connection Tab -> Go to filename and dynamic content:
@{concat(dataset().tablename,'.parquet')}
- Publish All
- You can use Debug or Add trigger to run the pipleline, go add trigger and click trigger now, click ok.
- Monitor it using monitor tab.
- Go to Azure Portal -> Data Storage -> Containers -> Layer 1, you can find SalesLT folder here which has been copied by the pipeline run.

------------------------------------------------------------------------------------------------------

Part 4: Data Transformation on Azure Databricks

- Go to Azure Databricks Service -> launch workspace
- Go to compute to create a compute cluster
- Rename to Data_Transform_Cluster
- Choose single mode -> Leave Databricks runtime as default
- Terminate after 15 minutes of inactivity to save money -> Create compute
- Go to workspace -> Workspace -> Shared -> Right click on canvas -> Create -> Notebook (will help to mount Azure Data Factory) -> Make sure language is python
- Rename notebook to mountADF
- Click connect at top right -> select Data_Transform_Cluster
- Go to Advanced Options -> Check Enable credential passthrough for user level access
- Go to notebook, use below link to help mount ADLS: using credential passthrough
https://learn.microsoft.com/en-us/azure/databricks/archive/credential-passthrough/adls-passthrough
- Get the python code from MS Learn website, and paste in notebook
- Replace <container-name> in the code with actual name, eg:
source = "abfss://layer-1@adlsgen2demo.dfs.core.windows.net/"
- Replace <mount-name> with storage account name, eg:
mount_point = "/mnt/layer-1"
- Check for green dot to check if cluster is connected
- Click Run all: check for output, should be "True".
- Write below code to test tables mounted earlier:
dbutils.fs.ls("/mnt/layer-1")
- Run it, if it returns operation failed, then appropriate permissions are not granted.
- Go to Azure Storage Account -> click on Access Control (IAM) -> Role Assignments -> Check your username -> Roles Tab -> Storage Blob Data Contributor -> Members -> Select Members -> Check your name -> Click Select - review + assign
- Get back to Databricks and Run second cell, check if it returns the tables list.
- Copy code form cell 1, change source and mount point form layer-1 to layer-2.
- To know what needs to be transformed, Go to MS SQL Server -> Open relevant table (Sales.T.Address)-> Transform dates from datetime format to date format
- Write and execute data transformation to all SalesLT tables
- Use query display(df) to check result, dates should be transformed
- To establish connection between azure databricks and azure data factory, go the pipeline, go to manage create a linked service, search for compute -> databricks, name it Databrickslinkedserv -> Select AutoresolveIntegrationRuntime -> select azure subscription and databricks workspace -> Use existing interactive cluster -> Click Access Token
- Go to Databricks -> User Settings -> Access Tokens -> Manage -> Generate New Token -> Generate with default options -> Copy and paste access token in Azure Kay Vault -> Secrets -> Create a secret -> Name: DBaccesstoken -> Secret Value: Paste Key -> Click create
- Go back to creating linked services -> Select Azure Key Vault Linked Service -> Fill all options -> Test Connection and create -> Publish All
- Go back to pipelines -> In activities, search for Notebook -> drag and drop databricks notebook to canvas -> change notebook name: layer-1 to layer-2.
- Go to Azure Databricks tab and select linked service
- Go to Settings tab and select notebook path (browse -> path -> layer-1 to layer-2 -> OK)
- Join the ForEach activity to Notebook activity by dragging the arrow on canvas
- Test Connection by using add trigger -> trigger now
- Go to monitor tab to monitor pipeline run and see if everything succeeded

------------------------------------------------------------------------------------------------------

Part 5: Data Loading

***Note: Azure Synapse analytics is a combination of Azure Data Factory and Azure Databricks. Anything possible with ADF and ADB can be achieved with Azure Synapse Analytics; Develop and Data are not there in ADF

- Go to Synapse Analytics Workspace -> Open Synapse Studio -> Data -> + Sign -> Create SQL Database -> Select Serverless (Only Compute/Lower Workloads), Dedicated is Both Compute and Storage/Higher Workloads.
- Name Database layer-2_db
- Under Linked Section, containers in Azure storage account/ADLS are already mounted by default
- Go to linked -> layer-2, CustomerAddress Table -> Select TOP 100 rows -> File Type = Delta Format
- A Query will open up, replace TOP 100 * with * to query whole table, switch use database from Master to later8_db and click RUN
- Run queries for all Tables so that they are listed under views section

------------------------------------------------------------------------------------------------------

Part 6: Data Reporting

- Launch PowerBI Desktop App
- Get Data -> Azure Synapse Analytics SQL
- Go get server URL, go Azure Synapse Workspace -> Properties -> Serverless SQL Endpoint -> Copy and Paste in PowerBI
- Name Database: layer2_db
- Data connectivity mode: Import
- Sign in with Microsoft Account Credentials
- Check all tables and click load
- Report View: Dashboards, Table View: Tables, Model View: Relationship between Tables
- Go to Relationship, PowerBI will have automatically created relationship between the tables
- New relationships can be created through drag and drop or by going to manage relationships
- Use PowerBI AI tool to create visual, double click the screen, ask AI tool a question to create report
- Use PowerBI Visualization tools to create remaining visuals

------------------------------------------------------------------------------------------------------

Part 7: End to End Pipeline Testing

- For end to end testing, go to ADF -> Add trigger -> New Trigger -> Name it trigger_schedule, Type: Schedule, Set Start Date and Time Zone
- Set Recurrence time to once a day (or as required)
- Check progress on monitor tab
- Refresh PowerBI to see results
- If PowerBI dashboard is updated, the pipeline works end to end.

------------------------------------------------------------------------------------------------------
