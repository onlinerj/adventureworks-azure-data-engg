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
SQL Server -> Azure Data Factory -> ADL Gen2 -> Azure Databricks -> Azure Synapse Analytics -> PowerBI

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
- 
- 
- 
- 

------------------------------------------------------------------------------------------------------

Part 5: Data Loading

- Go to Azure Databricks Service -> launch workspace
- 
- 
- 
- 

------------------------------------------------------------------------------------------------------

Part 6: Data Reporting

- 
- 
- 
- 
- 

------------------------------------------------------------------------------------------------------

Part 7: End to End Pipeline Testing

- 
- 
- 
- 
- 

