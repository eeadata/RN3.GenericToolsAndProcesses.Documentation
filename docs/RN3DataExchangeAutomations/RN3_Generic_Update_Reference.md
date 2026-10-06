(rn3-generic-update-reference)=
# RN3 Generic Update - Reference datasets

<hr class="double">

**Complexity:**  
<span class="stars">★★★★☆</span>


## Introduction

Reference dataset update is an FME-orchestrated process that updates selected RN3 dataflow **reference dataset tables** if the corresponding **reference data source** has changed since the last update. The reference data sources are tables or views in an **MS SQL database** (or databases).  

The process can be triggered manually or, more commonly, scheduled to run regularly.  

<hr class="thick">

## Quick setup

1. Ensure the **metadata harvesting process** is set up for the dataflows where you want to update reference data, and the **RN3 metadata tables** exist.  
     - See {ref}`RN3_Generic_Update_Reference.md-tables-rn3-dataflow-metadata-tables`.  
2. Choose a **Source database** and ask the EEA Service Desk to create it if it doesn't exist yet.  
    - See {ref}`RN3_Generic_Update_Reference.md-databases-source` database for more details.  
3. Create **Reference Data Source tables** in the Source database, one for each RN3 reference table you want to update.  
    - See {ref}`RN3_Generic_Update_Reference.md-tables-source-tables` for more details.  
4. Create the **Reference Update Tables table** in the RN3 database if it doesn't exist yet, and insert one record for each RN3 reference table you want to update.  
    - See How to {ref}`RN3_Generic_Update_Reference.md-tables-how-to-create-reference-update-tables-table` for an SQL example.  
    - See How to {ref}`RN3_Generic_Update_Reference.md-tables-how-to-populate-reference-update-tables-table` for an SQL example.  
    - See {ref}`RN3_Generic_Update_Reference.md-tables-reference-update-tables` for more details.  
5. Create the **Reference Update Parameters table** in the RN3 database if it doesn't exist yet, and insert one record for each RN3 dataflow where you want to update the reference dataset tables.  
    - See How to {ref}`RN3_Generic_Update_Reference.md-tables-how-to-create-reference-update-parameters-table` for an SQL example.  
    - See How to {ref}`RN3_Generic_Update_Reference.md-tables-how-to-populate-reference-update-parameters-table` for an SQL example.  
    - See {ref}`RN3_Generic_Update_Reference.md-tables-reference-update-parameters` for more details.  
6. **Create a data harvesting schedule** on the EEA's FME Flow server.  
	- See {ref}`RN3_Generic_Update_Reference.md-fme-workspace-schedule` for the details.  
	- See {ref}`RN3_Generic_Update_Reference.md-fme-workspace` for the details on the FME workspace.  

<hr class="thick">

## Requirements

The reference update setup process is complex and requires several steps to be completed correctly.  

The **prerequisite** for the process is the presence of metadata tables that are regularly updated by the **{ref}`rn3-generic-harvesting-metadata`** process.  

In addition to the **RN3 dataflow metadata** tables, the process uses additional SQL tables and views, some of which must be created and populated during the setup:  
- **reference data source** tables or views  
- table containing metadata of the individual **update jobs**  
- table containing list of **reference dataset tables** included in the update process and their data sources  
- table containing the required dataflow **reference update parameters**    

The tables don't need to be on the same MS SQL Server, but it's recommended they are, as it makes working with them easier.  

<hr class="thick">


## Databases

The data harvesting setup is flexible regarding the names and number of databases. The standard setup model, however, works with two databases.  
- **Source** database  
- **RN3** database  

*In this documentation, we will be using this model and the database names.*  

(RN3_Generic_Update_Reference.md-databases-source)=
### Source

This is a dataflow-specific database.  
In a standard EEA dataflow database model, the Production database usually serves as the Source database, but depending on the actual source of the reference data, it may also be the Import database, the 'non-suffix' database, or another database.  
Individual RN3 reference tables from the same dataset may even have their source data located in multiple different databases.  

The Source database contains the following **tables** involved in the reference update process:  
- **Reference Data Source** tables or views  
	- Must be created manually.  
- **Update Jobs** table  
	- Is created automatically.  

### RN3

This is a generic, server-wide metadata database.  
In the standard setup, this metadata database contains the following **tables** involved in the data reference update process:  
- **RN3 Dataflow Metadata** tables  
	- Must be present.  
- **Reference Update Tables** table  
	- Must be created manually if it doesn't exist yet.  
- **Reference Update Parameters** table  
	- Must be created manually if it doesn't exist yet.  

Because metadata harvesting is a prerequisite for the reference dataset update, this database should already be present when we start setting up the data harvesting process.  
See Metadata harvesting {ref}`RN3_Generic_Harvesting_Metadata.md-database` for more details.  

<hr class="thick">


## Tables

This documentation uses standard names for tables and schemas involved in the data harvesting process. The responsible data manager may, however, use different names if needed.  


(RN3_Generic_Update_Reference.md-tables-source-tables)=
### Reference Data Source tables

In the standard model, the Reference Data **Source tables** or views are placed in the **Source database**.  

Creating the Source tables should be the first step in setting up the reference dataset update process.  
Each RN3 reference dataset table we want to update must have a Source table or view in the database.  

The Source tables must include all data that the reference dataset tables need.  

<ins>**Schema and names**</ins>  

There is no standard convention for naming the Source tables. The data manager should use names and schemas that best fit their reference data management approach and naming conventions.  
However, the name should help identify the target RN3 reference table.  

It's also up to the data manager whether the reference source data is provided in a dedicated **Table** or 'produced on the fly' by a **View**.  

<ins>**Structure**</ins>

The Source table structure should match that of the target **RN3 reference dataset table**. The table columns must have the same names as the RN3 table fields (including the same letter case).  

In addition to the data columns, each Source table must include a *datetime* column with the timestamp of when the specific record was created or last updated.  
This column is used to check whether the reference data source has changed since the last update of the corresponding RN3 reference dataset table.  


(RN3_Generic_Update_Reference.md-tables-source-tables-reference)=
#### Reference

Each Source table or view must include a datetime column that includes the timestamp of the last reference record update.
The data manager may choose the name of this column, but we will use **lastUpdate** in the following text.  


(RN3_Generic_Update_Reference.md-tables-source-tables-explanation)=
#### Explanation

- The individual **Source tables or views** should be created in cooperation with the **responsible dataflow owner** (e.g., data steward).  
- The update process will fail if a Source table or view misses the **lastUpdate column**, or if it is empty for all records.  
- The update process will not fail if a Source table or view contains misnamed data columns or is missing columns for any RN3 table field. The process will not use misnamed columns, and RN3 table fields with missing Source columns will be empty after the update.  
- Each data column in the Source table or view should contain only data that are appropriate for the corresponding RN3 fields. The update process will not fail if the data type of the Source data values is incorrect, but discrepancies will most likely cause validation errors or failures in the RN3 dataflow.  
	
	```{important}
	If the *RN3 reference dataset table* contains any **geometry fields**, the corresponding Source table or view needs to contain the geometry values formatted as **Extended GeoJSON** strings, not as SQL geometry!  
	```
- The update compares the highest timestamp value in the **lastUpdate** column of the Source table with the highest **[tableUpdated]** value of the corresponding **Update Jobs table** records.  
    - If the *lastUpdate* value is higher, the process selects all data from the Source table and uses it to update the corresponding RN3 reference dataset table.  
	- If the *lastUpdate* value is lower, the process will not update the corresponding RN3 reference dataset table.  
	- If no Source table has a higher *lastUpdate* value, the update process ends there.  

<hr>

(RN3_Generic_Update_Reference.md-tables-update-jobs)=
### Update Jobs table

In the standard model, the Update Jobs table is placed in the **Source database**.  

The table contains a list of the update jobs and their metadata.  

The process uses the table to identify the latest update for the corresponding RN3 reference dataset table. At the end of the update, it adds a new record for each reference table it updated, or attempted to update, and documents the job result.  

<ins>**Schema and name**</ins>  

The standard table name is **[ReferenceUpdateJobs]** and the standard schema is **[metadata]**.  

When the update process runs for a specific dataflow for the first time, it creates the schema and the table if they don't exist yet.  

```{warning}
It's not recommended to create the Update Jobs table manually.  
```

<ins>**Structure**</ins>  

The Update Jobs table structure includes metadata columns needed to uniquely identify each job at the reference table level, as well as each job's result.  


(RN3_Generic_Update_Reference.md-tables-update-jobs-reference)=
#### Reference

The Harvesting Jobs table contains these columns:  
- **[dataflowId]** - The RN3 dataflow identifier.  
- **[datasetId]** - The RN3 reference datset identifier.  
- **[datasetName]** - The RN3 reference dataset name.  
- **[tableSchemaId]** - The RN3 reference dataset table schema identifier.  
- **[tableName]** - The RN3 reference dataset table name.  
- **[tableUpdated]** - A timestamp when the reference table was successfully updated.  
- **[jobId]** - An identifier of the FME job that ran the reference update process.  
- **[jobExecuted]** - A timestamp of when the FME job started. 
- **[jobSummary]** - A summary of the reference update job.  

(RN3_Generic_Update_Reference.md-tables-update-jobs-explanation)=
#### Explanation

- At the beginning, the reference update process finds the highest timestamp value in the **lastUpdate** column of each reference data source table involved in the update. It then compares it with the highest **[tableUpdated]** timestamp value of matching Update Jobs table records. If no matching records exist, or if the **lastUpdate** value is higher, the process selects data from the source table and attempts to update the RN3 reference table.  
- Whether the import succeeds or fails, the process enters a new record in the Updates Jobs table. The **[tableUpdated]** field is filled only if the import succeeds.  
- If the import was successful, the **[jobSummary]** will include a short confirmation of the success. If it failed, the field will include detailed information. In both cases, the field also includes the highest **lastUpdate** timestamp of the specific reference data source.  
 

<hr>

(RN3_Generic_Update_Reference.md-tables-rn3-dataflow-metadata-tables)=
### RN3 Dataflow Metadata tables

The RN3 Dataflow Metadata tables are described in the **{ref}`RN3_Generic_Harvesting_Metadata.md-tables`** section of the **RN3 Generic Harvesting - metadata** document.  

The tables must exist and be populated for the update process to function.  
 
The reference update process uses the following RN3 metadata tables:  
- **[Dataflow]**  
- **[DesignDataset]**  
- **[ReferenceDataset]**  

The **[Dataflow]** table provides the RN3 dataflow ApiKey value the process uses to authorise the data import request.  

The **[DesignDataset]** table is the source of the RN3 dataset schema, including the structure of the individual tables.  

The **[ReferenceDataset]** table provides information on the reference datasets included in the specific RN3 dataflow.  

<hr>

(RN3_Generic_Update_Reference.md-tables-reference-update-tables)=
### Reference Update Tables table

In the standard model, the Reference Update Tables table is placed in the **RN3 database**.  

It contains a list of **RN3 reference dataset tables** to be updated by the process and parameters of the corresponding **Reference Data Source** tables.  

<ins>**Schema and name**</ins>  

The standard table name is **[ReferenceUpdateTables]** and the standard schema is **[metadata]**.  

<ins>**Structure**</ins>  

The [RN3].[metadata].[ReferenceUpdateTables] table contains columns for three parameter groups:  

1. **Dataflow-related** parameters  
	- [dataflowId], [dataflowName], [datasetId], [datasetName]  

2. **RN3 table** parameters
	- [table_RN3_Name], [table_RN3_SchemaID],[table_RN3_update]  

3. **Reference Data Source table** parameters  
	- [dataSource_DatabaseConnection], [dataSource_Database], [dataSource_Schema], [dataSource_Name], [dataSource_Field_lastUpdate]  

(RN3_Generic_Update_Reference.md-tables-reference-update-tables-reference)=
#### Reference

[RN3].[metadata].[ReferenceUpdateTables] table columns:  
- **[dataflowId]**  - The RN3 dataflow identifier.  
- **[dataflowName]**  - Optional. The name of the RN3 dataflow.  
- **[datasetId]** - The RN3 reference dataset identifier.  
- **[datasetName]** - The name of the RN3 reference dataset.  
- **[table_RN3_Name]** - The name of the RN3 reference table.  
- **[table_RN3_SchemaID]** - The schema identifier of the RN3 reference table.  
- **[table_RN3_update]** - A boolean field indicating whether the RN3 reference table should be updated or not (1 = yes, 0 = no).  
- **[dataSource_DatabaseConnection]** - Name of the FME Database connection linked to the MS SQL database containing the Reference Data Source table.  
- **[dataSource_Database]** - Name of the MS SQL database containing the Reference Data Source table.   
- **[dataSource_Schema]** - Name of the Reference Data Source table schema.  
- **[dataSource_Name]** - Name of the Reference Data Source table.    
- **[dataSource_Field_lastUpdate]** - Name of the datetime field in the Reference Data Source table that contains timestamp values of the record's last update.  

(RN3_Generic_Update_Reference.md-tables-reference-update-tables-explanation)=
#### Explanation

- If the [RN3].[metadata].[ReferenceUpdateTables] table doesn't exist yet, a responsible data manager must create it.  
	```{seealso}
	See How to {ref}`RN3_Generic_Update_Reference.md-tables-how-to-create-reference-update-tables-table` for an example.  
	```
- The data manager needs to add a record for each RN3 reference table they want to update and provide all mandatory values.  
	```{seealso}
	See How to {ref}`RN3_Generic_Update_Reference.md-tables-how-to-populate-reference-update-tables-table` for an example.
	```
- It is in this table that the data manager specifies the source of the data for the RN3 reference table, its name and location.  

```{warning}
Be careful not to delete the [RN3].[metadata].[ReferenceUpdateParameters] table if it already exists, or delete or change records for other dataflows that use the same table.  
```

- The value for the **table_RN3_SchemaID** field can be extracted from the RN3 dataset URL when a specific RN3 table is selected. It is the value of the 'tab' parameter.   

	```{admonition} Example
	:class: dropdown
	***URL:*** https://sandbox.reportnet.europa.eu/dataflow/13110/datasetSchema/46012?tab=6a452708c2c277000148527d  
	***rn3_tableSchemaId:*** 6a452708c2c277000148527d  
	```

- The FME database connection referred to in the **dataSource_DatabaseConnection** must exist in the EEA's FME Flow server. The connection must be JDBC, and its name should end with the suffix '_JDBC'. The FME Flow server needs at least *db_reader* permission on the linked database to access it.  

    ```{seealso}
	How to {ref}`RN3_Generic_Update_Reference.md-how-to-create-database-connection`  
	How to {ref}`RN3_Generic_Update_Reference.md-how-to-add-fme-database-user`  
	``` 

- The database connection parameters may contain both non-JDBC and JDBC versions of the connection name. The non-JDBC connection names must match the JDBC one, just without the '_JDBC' suffix. The data harvesting process automatically creates attributes with the JDBC versions of the database connections if the respective parameters contain non-JDBC ones.  

- The  **[dataSource_Field_lastUpdate]** value must match the **lastUpdate** column name in the corresponding **Reference Data Source table**, otherwise the update process will fail.  

#### How to

(RN3_Generic_Update_Reference.md-tables-how-to-create-reference-update-tables-table)=
##### Create [metadata].[ReferenceUpdateTables] table

**SQL - example:**  

~~~~sql
--DROP TABLE [RN3].[metadata].[ReferenceUpdateTables]

CREATE TABLE [RN3].[metadata].[ReferenceUpdateTables](
	[dataflowId] [bigint] NOT NULL,
	[dataflowName] [nvarchar](255) NULL,
	[datasetId] [bigint] NOT NULL,
	[datasetName] [nvarchar](255) NOT NULL,
	[table_RN3_Name] [nvarchar](255) NOT NULL,
	[table_RN3_SchemaID] [nvarchar](255) NOT NULL,
	[table_RN3_update] [bit] NOT NULL,
	[dataSource_DatabaseConnection] [nvarchar](255) NOT NULL,
	[dataSource_Database] [nvarchar](255) NOT NULL,
	[dataSource_Schema] [nvarchar](255) NOT NULL,
	[dataSource_Name] [nvarchar](255) NOT NULL,
	[dataSource_Field_lastUpdate] [nvarchar](255) NOT NULL,
PRIMARY KEY CLUSTERED 
(
	[datasetId] ASC,
	[table_RN3_Name] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON, OPTIMIZE_FOR_SEQUENTIAL_KEY = OFF) ON [PRIMARY]
) ON [PRIMARY]
~~~~

(RN3_Generic_Update_Reference.md-tables-how-to-populate-reference-update-tables-table)=
##### Populate [metadata].[ReferenceUpdateTables] table

**SQL - example:**  

~~~~sql
--DELETE
--SELECT *
--FROM [RN3].[metadata].[ReferenceUpdateTables]
--WHERE [dataflowId] IN (<dataflowId>,<dataflowId>)

INSERT INTO [RN3].[metadata].[ReferenceUpdateTables]
    ([dataflowId]
    ,[dataflowName]
    ,[datasetId]
    ,[datasetName]
    ,[table_RN3_Name]
    ,[table_RN3_SchemaID]
    ,[table_RN3_update]
    ,[dataSource_DatabaseConnection]
    ,[dataSource_Database]
    ,[dataSource_Schema]
    ,[dataSource_Name]
    ,[dataSource_Field_lastUpdate])
VALUES

-- one record for each RN3 reference table that should be updated
    ( <dataflowId>,'<dataflowName>',<datasetId>,'<datasetName>','<rn3_tablename>','<rn3_schemaId>',1,'<Source_database_FME_connection_name>','<Source_database_name','<source_table_schema>','<source_table_name>','lastUpdate' )
    ,( <dataflowId>,'<dataflowName>',<datasetId>,'<datasetName>','<rn3_tablename>','<rn3_schemaId>',1,'<source_database_FME_connection_name>','<source_database_name','<source_table_schema>','<source_table_name>','lastUpdate' )
~~~~

<hr>

(RN3_Generic_Update_Reference.md-tables-reference-update-parameters)=
### Reference Update Parameters table

In the standard model, the Reference Update Parameters table is placed in the **RN3 database**.  

It contains the core parameters used by the reference dataset update process, one record for each RN3 dataflow where we want to update the reference dataset tables.  
The majority of the parameters identify the other metadata tables (Update Jobs, RN3 metadata, Reference Update Tables) the process must use to update the reference tables.  

 <ins>**Schema and name**</ins>  

The standard table name is **[ReferenceUpdateParameters]** and the standard schema is **[metadata]**.  

<ins>**Structure**</ins>  

The [RN3].[metadata].[ReferenceUpdateParameters] table contains columns for four parameter groups:  

1. **Dataflow-related** parameters  
	- [dataflowId], [dataflowName]  

2. **RN3 metadata tables** parameters
	- [metadata_DatabaseConnection] ,[metadata_Database] ,[metadata_Schema] ,[metadata_Table_Dataflow] ,[metadata_Table_ReferenceDataset] ,[metadata_Table_DesignDataset]

3. **Reference Update Tables table** parameters  
	- [referenceTables_DatabaseConnection] ,[referenceTables_Database] ,[referenceTables_Schema] ,[referenceTables_Table]

4. **Update Jobs table** parameters  
	- [updateJobs_DatabaseConnection] ,[updateJobs_Database] ,[updateJobs_Schema] ,[updateJobs_Table]


(RN3_Generic_Update_Reference.md-tables-reference-update-parameters-reference)=
#### Reference

- **[dataflowId]**  - The RN3 dataflow identifier.  
- **[dataflowName]**  - Optional. The name of the RN3 dataflow.  
- **[metadata_DatabaseConnection]** - Name of the FME Database connection linked to the MS SQL database containing the RN3 metadata tables.  
- **[metadata_Database]** - Name of the MS SQL database containing the RN3 metadata tables (standard is 'RN3').  
- **[metadata_Schema]** - Name of the RN3 metadata tables schema (standard is 'metadata').  
- **[metadata_Table_Dataflow]** - Name of the table containing RN3 dataflow metadata (standard is 'Dataflow').  
- **[metadata_Table_ReferenceDataset]** - Name of the table containing RN3 reference dataset metadata (standard is 'ReferenceDataset').  
- **[metadata_Table_DesignDataset]** - Name of the table containing RN3 design dataset metadata (standard is 'DesignDataset').  
- **[referenceTables_DatabaseConnection]** - Name of the FME Database connection linked to the MS SQL database containing the Reference Update Tables table.  
- **[referenceTables_Database]** - Name of the MS SQL database containing the Reference Update Tables table (standard is 'RN3').  
- **[referenceTables_Schema]** - Name of the Reference Update Tables table schema (standard is 'metadata').  
- **[referenceTables_Table]** - Name of the Reference Update Tables table (standard is 'ReferenceUpdateTables').  
- **[updateJobs_DatabaseConnection]** - Name of the FME Database connection linked to the MS SQL database containing the Update Jobs table.  
- **[updateJobs_Database]** - Name of the MS SQL database containing the Update Jobs table (standardly it is the dataflow-specific Production, Import or non-suffix database).  
- **[updateJobs_Schema]** - Name of the Update Jobs table schema (standard is 'metadata'). 
- **[updateJobs_Table]** - Name of the Update Jobs table (standard is 'UpdateJobs').

(RN3_Generic_Update_Reference.md-tables-reference-update-parameters-explanation)=
#### Explanation

- If the [RN3].[metadata].[ReferenceUpdateParameters] table doesn't exist yet, a responsible data manager must create it.
	```{seealso}
	See How to {ref}`RN3_Generic_Update_Reference.md-tables-how-to-create-reference-update-parameters-table` for an example.
	```
- The data manager needs to add a record for each RN3 dataflow where they want to update the reference data and provide all mandatory values.
	```{seealso}
	See How to {ref}`RN3_Generic_Update_Reference.md-tables-how-to-populate-reference-update-parameters-table` for an example.
	```
- It is in this table that we specify the databases, table schemas and names of metadata tables, whether they follow the standard or differ from it.  

```{warning}
Be careful not to delete the [RN3].[metadata].[ReferenceUpdateParameters] table if it already exists, or delete or change records for other dataflows that use the same table.  
```

- The FME database connections referred to in the **metadata_DatabaseConnection**, **referenceTables_DatabaseConnection** and **updateJobs_DatabaseConnection** parameters must exist in the EEA's FME Flow server. The connections must be **JDBC**, and their names should end with the suffix '_JDBC'. The FME Flow server needs at least *db_reader* permissions on the linked databases to read the metadata tables.  

    ```{seealso}
	How to {ref}`RN3_Generic_Update_Reference.md-how-to-create-database-connection`  
	How to {ref}`RN3_Generic_Update_Reference.md-how-to-add-fme-database-user`  
	``` 

- The database connection parameters may contain both non-JDBC and JDBC versions of the connection name. The non-JDBC connection names must match the JDBC one, just without the '_JDBC' suffix. The data harvesting process automatically creates attributes with the JDBC versions of the database connections if the respective parameters contain non-JDBC ones.  


#### How to

(RN3_Generic_Update_Reference.md-tables-how-to-create-reference-update-parameters-table)=
##### Create [metadata].[ReferenceUpdateParameters] table

**SQL - example:**  

~~~~sql
CREATE TABLE [RN3].[metadata].[ReferenceUpdateParameters](
	[dataflowId] [bigint] NOT NULL,
	[dataflowName] [nvarchar](255) NULL,
	[metadata_DatabaseConnection] [nvarchar](255) NOT NULL,
	[metadata_Database] [nvarchar](255) NOT NULL,
	[metadata_Schema] [nvarchar](255) NOT NULL,
	[metadata_Table_Dataflow] [nvarchar](255) NOT NULL,
	[metadata_Table_ReferenceDataset] [nvarchar](255) NOT NULL,
	[metadata_Table_DesignDataset] [nvarchar](255) NOT NULL,
	[referenceTables_DatabaseConnection] [nvarchar](255) NOT NULL,
	[referenceTables_Database] [nvarchar](255) NOT NULL,
	[referenceTables_Schema] [nvarchar](255) NOT NULL,
	[referenceTables_Table] [nvarchar](255) NOT NULL,
	[updateJobs_DatabaseConnection] [nvarchar](255) NOT NULL,
	[updateJobs_Database] [nvarchar](255) NOT NULL,
	[updateJobs_Schema] [nvarchar](255) NOT NULL,
	[updateJobs_Table] [nvarchar](255) NOT NULL
) ON [PRIMARY]

~~~~

(RN3_Generic_Update_Reference.md-tables-how-to-populate-reference-update-parameters-table)=
##### Populate [metadata].[ReferenceUpdateParameters] table

**SQL - example:**  

~~~~sql
INSERT INTO [RN3].[metadata].[ReferenceUpdateParameters]
    ([dataflowId]
    ,[dataflowName]

    ,[metadata_DatabaseConnection]
    ,[metadata_Database]
    ,[metadata_Schema]
    ,[metadata_Table_Dataflow]
    ,[metadata_Table_ReferenceDataset]
    ,[metadata_Table_DesignDataset]

    ,[referenceTables_DatabaseConnection]
    ,[referenceTables_Database]
    ,[referenceTables_Schema]
    ,[referenceTables_Table]

    ,[updateJobs_DatabaseConnection]
    ,[updateJobs_Database]
    ,[updateJobs_Schema]
    ,[updateJobs_Table])
VALUES
     -- Dataflow name
    (
    <dataflowId>, '<dataflowName>'

    , '<RN3_database_FME_connection_name>'
    , '<RN3_database_name>'
    , 'metadata', 'Dataflow', 'ReferenceDataset', 'DesignDataset'
    
    , '<RN3_database_FME_connection_name>'
    , '<RN3_database_name>'
    , 'metadata', 'ReferenceUpdateTables'
    
    , '<Source_database_FME_connection_name>'
    , '<Source_database_name>'
    , 'metadata', 'UpdateJobs'
    ),

-- Dataflow name
    (
    <dataflowId>, '<dataflowName>'

    , '<RN3_database_FME_connection_name>'
    , '<RN3_database_name>'
    , 'metadata', 'Dataflow', 'ReferenceDataset', 'DesignDataset'
    
    , '<RN3_database_FME_connection_name>'
    , '<RN3_database_name>'
    , 'metadata', 'ReferenceUpdateTables'
    
    , '<Source_database_FME_connection_name>'
    , '<Source_database_name>'
    , 'metadata', 'UpdateJobs'
    )

~~~~


<hr class="thick">


(RN3_Generic_Update_Reference.md-fme-workspace)=
## FME workspace 

The reference data update FME workspace does the following: 
- Processes user parameters and selects relevant records from the **Reference Update Parameters** table.  
- Creates JDBC versions of database connections if the parameters provide non-JDBC versions.  
- Selects relevant records from the **Reference Update Tables** table.  
- Creates the **Update Jobs** table schema if it doesn't exist yet.  
- Creates the **Update Jobs** table if it doesn't exist yet.  
- Selects the highest **lastUpdate** value from each **Reference Data Source table** and compares it with the the highest **[tableUpdated]** value of the corresponding **Update Jobs table** records.  
- Selects all records from the **Reference Data Source tables** where the highest **lastUpdate** value is higher.  
- Parses JSON schema of the corresponding RN3 tables selected from the RN3 metadata table **[DesignDataset]**, and converts it to dynamic schema for CSV files.   
- Creates CSV files using the source data and the dynamic schema.  
- Selects dataflow API key from the RN3 metadata table **[Dataflows]**.  
- Zips the CSV files.  
- Sends ZIP file to Reportnet 3 using the */dataset/v2/importFileData/{datasetId}* API endpoint, with parameter replace=true.  
- Parses the response, checks the status of the RN3 job and waits until it finishes (or fails).
- Inserts a record into the **Update Jobs table** with the jobs summary.

**Latest version:**  
<https://fme.discomap.eea.europa.eu/fmeserver/workspaces/run/Dataflows_RN3_Generic_Processes/RN3_Generic_Update_referenceData.fmw>

(RN3_Generic_Update_Reference.md-fme-workspace-user-parameters)=
### User parameters

#### Reference

**Mandatory parameters:**  
- **dataflowIds** - A comma-separated list of all RN3 dataflows where the reference dataset tables should be updated.  
- **UP_databaseConnection** - Name of the FME Database connection linked to the MS SQL database containing the Reference Update Parameters table. It must be a **JDBC** MSSQL connection.
 - **UP_database** - Name of the MS SQL database containing the Reference Update Parameters table (standard is 'RN3'). 
 - **UP_schema** - Name of the Reference Update Parameters table schema (standard is 'metadata'). 
 - **UP_table** - Name of the Reference Update Parameters table (standard is 'ReferenceUpdateParameters').
- **baseUrl** - The base URL of the specific Reportnet 3 platform's API service (e.g., https://api.reportnet.europa.eu).

#### Explanation

- In the **UP_** parameters we specify the name and location of the **Reference Update Parameters table**. This is where we can identify whether the location, schema or name is non-standard.
- If the database connection referred to in the **UP_databaseConnection** parameter is not a JDBC connection, the harvesting process will fail.  
- The FME connection referred to in the **UP_databaseConnection** parameter must exist in the EEA's FME Flow server. The connection must be non-JDBC. The FME Flow server needs *db_ddladmin* or *db_owner* permissions on the linked database to access it and write to the metadata tables.  

    ```{seealso}
	How to {ref}`RN3_Generic_Update_Reference.md-how-to-create-database-connection`  
	How to {ref}`RN3_Generic_Update_Reference.md-how-to-add-fme-database-user`  
	``` 
<hr>

(RN3_Generic_Update_Reference.md-fme-workspace-schedule)=
### Schedule

The reference data update processes are commonly run on a regular schedule. This can be achieved by creating an FME Schedule (or Automation) that triggers the FME workspace at specified intervals and supplies it with corresponding user parameters.  
The recommended frequency for reference data updates is once a day. It's highly recommended to run it late at night.  
The generic metadata harvesting should run about 10-15 minutes before the data harvesting so it has time to finish.

```{seealso}
How to {ref}`RN3_Generic_Update_Reference.md-how-to-create-fme-schedule`  
``` 

<hr>

### How to

(RN3_Generic_Update_Reference.md-how-to-create-database-connection)=
#### Create a database connection on FME Flow

- Follow <https://support.safe.com/hc/en-us/articles/25407463461517-FAQ-Database-Connections-on-FME-Flow>.
- If you don't have permissions to add a connection, or are not confident enough to create it, ask EEA Service Desk to create it for you.

(RN3_Generic_Update_Reference.md-how-to-add-fme-database-user)=
#### Add FME Flow server as a user to an MS SQL database

- Follow <https://learn.microsoft.com/en-us/sql/relational-databases/security/authentication-access/create-a-database-user?view=sql-server-ver17#create-a-user-with-ssms>.
- Add a user with eeadmz1\fmeservice as the user name and login name.
- Select appropriate permission options on the Membership page.
- If you don't have permissions to add users to the database, ask EEA Service Desk to add it.

(RN3_Generic_Update_Reference.md-how-to-create-fme-schedule)=
#### Create a Schedule on FME Flow.
- Follow <https://docs.safe.com/fme/html/FME-Flow/WebUI/schedules.htm>.
- If you don't have permissions to create a Schedule, or are not confident enough to create it, ask the EEA Service Desk or a colleague with appropriate permissions and experience to create it for you.
