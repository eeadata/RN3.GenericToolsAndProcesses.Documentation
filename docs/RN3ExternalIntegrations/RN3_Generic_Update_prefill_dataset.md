# RN3 Generic Update - prefill dataset

___

**Description:**  
This external integration adds an Import dataset data option to the RN3 dataset, allowing the data provider to prefill the whole dataset with selected data from an MS SQL database (or databases).  

It requires the presence of a custom SQL table - ***prefill parameters table***. This table contains:
- references of the RN3 dataset tables that the process should prefill (***target tables***),  
- the corresponding ***prefill data sources*** - tables or views in an MS SQL database(s) containing prefill data,  
- and other pertinent information.  

The external integration FME workspace accesses the relevant information from the specified *prefill parameters table*, uses it to read data from the referred *prefill data sources*, applies specified filters if provided (usually the data provider code), and imports the results into the *target tables*.  

>**WARNING**  
>Any existing records in the *target tables* will be automatically deleted before the import.  

The process transforms and saves the selected *prefill data sources* data into a series of CSV files (one per target table), zips the CSV files into one ZIP file, which it then imports to the RN3 reporting dataset using the '/dataset/v2/importFileData/{datasetId}' RN3 API endpoint.  

>**WARNING**  
>The process doesn't check if the selected *prefill data sources* data match the constraints of the *target tables* fields (e.g., data type) before the import. The discrepancies will be identified during the RN3 Validation.  

**Operation type:**  
IMPORT FROM OTHER SYSTEM  

**Latest version:**  
<https://fme.discomap.eea.europa.eu/fmeserver/workspaces/run/Dataflows_RN3_Generic_Processes/RN3_Generic_Update_prefillDataset.fmw>


## External integration setup - example

**Name:**  
Prefill dataset  

**Description:**  
Prefill dataset (generic)  

**Repository:**  
Dataflows_RN3_Generic_Processes  

**Workspace name:**  
RN3_Generic_Update_prefillDataset.fmw  

**Operation:**  
IMPORT FROM OTHER SYSTEM  

**Custom parameters**  

| **Parameter key** | **Parameter value** |
| --- | --- |
| **DPP_databaseConnection** | *WISE_SOE_Production_WIGEON* |
| **DPP_databaseName** | *WISE_SOE_Production* |
| **DPP_tableSchema** | *metadata* |
| **DPP_tableName** | *RN3_DatasetPrefillParameters* |


## External integration custom parameters

### Reference

**Mandatory parameters:**  
- **DPP_databaseConnection** - Name of the FME Database connection linked to the MS SQL database containing the *prefill parameters table*.  
- **DPP_databaseName** - Name of the MS SQL database containing the *prefill parameters table*.  
- **DPP_tableSchema** - Name of the table schema of the *prefill parameters table*.  
- **DPP_tableName** - Name of the table or view that contains the dataset prefill parameters. The ***prefill parameters table***.  

### Explanation

- The FME connection referred to in the **DPP_databaseConnection** parameter must exist in the EEA's FME Flow server. The connection can be either non-JDBC or JDBC. If it's JDBC, the connection name must end with '_JDBC'. The FME Flow server needs at least *db_datareader* permissions in the linked database to access and read the *prefill parameters table*. 
    > See {ref}`RN3_Generic_Update_prefill_dataset.md-how-to-1` for more details.

(RN3_Generic_Update_prefill_dataset.md-how-to-1)=
### How to

#### Create a database connection on FME Flow

- Follow <https://support.safe.com/hc/en-us/articles/25407463461517-FAQ-Database-Connections-on-FME-Flow>
- If you don't have permissions to add a connection, or are not confident enough to create it, ask EEA Service Desk to create it for you

#### Add FME Flow server as a user to an MS SQL database

- Follow <https://learn.microsoft.com/en-us/sql/relational-databases/security/authentication-access/create-a-database-user?view=sql-server-ver17#create-a-user-with-ssms>
- Add a user with eeadmz1\fmeservice as the user name and login name
- Select appropriate permission options in the Membership page
- If you don't have permissions to add users to the database, ask EEA Service desk to add it

## Prefill parameters table

This is an MS SQL Server database table with a defined structure that should contain a record for each RN3 dataset table the process should prefill.  
It doesn't have to be located in the same database as the *prefill data sources*  

### Reference

**Table structure:**  

- **rn3_dataflowId** - RN3 dataflow ID.  
- **rn3_datasetName** - RN3 dataset name.  
- **rn3_tableName** - Name of the RN3 dataset table to be prefilled. The **target table**.  
- **rn3_tableSchemaId** - RN3 table schema ID.  
- **rn3_tablePrefill** - A boolean field indicating if the RN3 table should be prefilled (1) or not (0).  
- **ps_databaseConnection** - Name of the FME database connection linked to the MS SQL database containing the *table prefill data source*.  
- **ps_databaseName** - Name of the MS SQL database containing the *table prefill data source*.  
- **ps_tableSchema** - Name of the table schema of the *table prefill data source*.  
- **ps_tableName** - Name of the table or view that contains the table's prefill data. The ***table prefill data source***.  
- **ps_fieldName_provider** - Name of a field in the *table prefill data source*, which contains the data provider code values.  

### Explanation
- The value for the **rn3_tableSchemaId** field can be extracted from the RN3 dataset URL when a specific RN3 table is selected. It is the value of the 'tab' parameter.   

	> **EXAMPLE**  
	> ***URL:*** https://sandbox.reportnet.europa.eu/dataflow/13110/datasetSchema/46012?tab=6a452708c2c277000148527d  
	> ***rn3_tableSchemaId:*** 6a452708c2c277000148527d  

- The individual ***table prefill data sources*** should be created in cooperation with the **responsible dataflow owner** (e.g., data steward).
- The FME connection referred to in the **ps_databaseConnection** field must exist in the EEA's FME Flow server. The connection can be either non-JDBC or JDBC. If it's JDBC, the connection name must end with '_JDBC'. The FME Flow server needs at least *db_datareader* permissions in the linked database to access and read the *table prefill data source*. 
- The process will import values only from those columns in the *table prefill data source* that have the **same name** as the fields in the *target table*. Any additional columns in the *table prefill data source* will be ignored. The *target table* fields that do not have a corresponding column in the *table prefill data source* will be imported empty. 
- If the *target table* contains any **geometry fields**, and the prefill action should prefill those too, the *table prefill data source* needs to contain the geometry values formatted as Extended GeoJSON string, not as SQL geometry! 

	> **NOTE**  
	> This may change in future versions of the FME workspace.  

- If the **ps_fieldName_provider** value is provided, it's used to select only those *table prefill data source* records where the value in the specified field is the same as the RN3 dataset's data provider code value (RN3 supplies the data provider code value to the FME workspace automatically, as the countryCode user parameter). If the *table prefill data source* doesn't contain a column with the name matching the **ps_fieldName_provider** value, the import process will fail.  
- If the **ps_fieldName_provider** value is not provided, the process selects and imports all records from the *table prefill data source* to the *target table*.  

### How to

#### Create prefill parameters table

**SQL - example:**  

~~~~sql
USE [WISE_SOE_Production]

-- DROP TABLE [metadata].[RN3_DatasetPrefillParameters]

CREATE TABLE [metadata].[RN3_DatasetPrefillParameters](
	[rn3_dataflowId] [bigint] NOT NULL,
	[rn3_datasetName] [nvarchar](255) NULL,	
	[rn3_tableName] [nvarchar](255) NOT NULL,
	[rn3_tableSchemaId] [nvarchar](255) NOT NULL,
	[rn3_tablePrefill] [bit] NOT NULL,
	[ps_databaseConnection] [nvarchar](255) NOT NULL,
	[ps_databaseName] [nvarchar](255) NOT NULL,
	[ps_tableSchema] [nvarchar](255) NOT NULL,
	[ps_tableName] [nvarchar](255) NOT NULL,
	[ps_fieldName_provider] [nvarchar](255) NULL
) ON [PRIMARY]
~~~~

#### Populate prefill parameters table
**SQL - example:**  

~~~~sql
USE [WISE_SOE_Production]

-- DELETE FROM [metadata].[RN3_DatasetPrefillParameters] WHERE [rn3_dataflowId] = 13110

INSERT INTO [metadata].[RN3_DatasetPrefillParameters]
    (
        [rn3_dataflowId]
        ,[rn3_datasetName]        
        ,[rn3_tableName]
        ,[rn3_tableSchemaId]
        ,[rn3_tablePrefill]
        ,[ps_databaseConnection]
        ,[ps_databaseName]
        ,[ps_tableSchema]
        ,[ps_tableName]
        ,[ps_fieldName_provider]
    )
VALUES
    (
        13110
        ,'1 - Reporting data'        
        ,'Emissions'
        ,'6a452708c2c277000148527d'
        ,1
        ,'WISE_SOE_Production_WIGEON'
        ,'WISE_SOE_Production'
        ,'rn3_datasetPrefill'
        ,'v_T_WISE1_Emissions'
        ,'countryCode'
    )
    ,
    (
        13110
        ,'1 - Reporting data'
        ,'RiverineInputLoads'
        ,'6a452712c2c277000148528a'
        ,1
        ,'WISE_SOE_Production_WIGEON'
        ,'WISE_SOE_Production'
        ,'rn3_datasetPrefill'
        ,'v_T_WISE1_RiverineInputLoads'
        ,'countryCode'
    )
~~~~