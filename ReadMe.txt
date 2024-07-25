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
- 
- 
- 
- 
- 
- 