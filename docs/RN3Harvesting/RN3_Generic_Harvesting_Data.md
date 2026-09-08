# RN3 Generic Harvesting - Data

___

**Complexity:**  
<span class="stars">★★★★⯪</span>


## Introduction

Data harvesting is an FME-orchestrated process that downloads **data** from **data collection** datasets of selected **Reportnet 3 dataflows** and stores it in a **series of tables** in an **MS SQL database**.

The process is designed to harvest only data that has not yet been successfully harvested.  
This usually means data released since the last harvesting, or where the harvesting has previously failed.  
The process can also be forced to reharvest previously harvested data.  

```{warning}
The process can harvest only data from the latest release. If a data provider made multiple releases between two harvesting jobs, the process won't harvest data from earlier releases.  
Older releases also can't be reharvested. Instead, the data will be replaced with data from the latest release.  

*This will change in the future, once the required changes are done to the RN3 platform to allow harvesting of specific release snapshots.*  
```

The process harvests **geometry data** and can be configured to convert it into SQL geometry.  
It can also be configured to download the **attachment files** and store them in a specified folder.  

The process can be triggered manually or, more commonly, scheduled to run regularly.  

## Requirements

The data harvesting setup process is complex and requires several steps to be completed correctly.  

The **prerequisite** for the process is the presence of metadata tables that are regularly updated by the **{ref}`rn3-generic-harvesting-metadata`** process.  

In addition to the **RN3 dataflow metadata** tables, the process uses additional SQL tables, some of which must be created and populated during the setup:  
- tables to store **harvested data**  
- harvested data **template** tables  
- table containing metadata of the individual **harvesting jobs**  
- table containing list of **dataflow tables** included in the harvesting process  
- table containing a list of **geometry fields**, used to specify how the geometry data should be harvested
- table containing the required dataflow **harvesting parameters**  

With the exception of the *templates* tables, all other tables must be located in databases on the same MS SQL Server.  

## Databases

The data harvesting setup is flexible regarding the names and number of databases. The standard setup model, however, works with two databases.  
- **Import** database  
- **RN3** database  

*In this documentation, we will be using this model and the database names.*  

### Import

This is a dataflow-specific database.  
In a standard EEA dataflow database model, the Import database name consists of the abbreviation of the dataflow or the dataflow group, and the suffix '_Import' (e.g., NatDA_Import, WISE_SoE_Import).
The data manager may use a different database, or even multiple databases if needed.
  
The Import database contains the following **tables** involved in the data harvesting process:  
- **Template** tables  
- **Harvested data** tables  
- **Harvesting jobs** table  

### RN3

This is a generic, server-wide metadata database.  
In the standard setup, this metadata database contains the following **tables** involved in the data harvesting process:  
- **RN3 dataflow metadata** tables  
- **Dataflow tables** table  
- **Geometry fields** table  
- **Harvesting parameters** table  

Because metadata harvesting is a prerequisite for data harvesting, this database should already be present when we start setting up the data harvesting process.  
See Metadata harvesting {ref}`RN3_Generic_Harvesting_Metadata.md-database` for more details.  


## Tables

This documentation uses standard names for tables and schemas involved in the data harvesting process. The responsible data manager may, however, use different names if needed.


(RN3_Generic_Harvesting_Data.md-tables-templates)=
### Template tables

Creating template tables should be the first step in the setup.  
Each RN3 dataflow table that should have its data harvested must have a template table in the Import database.  

The template tables must be left empty.  

The harvesting process uses the template tables as the source of dynamic schemas when writing the data to the database.  
When it runs for the first time for a specific dataflow, it also uses them to create the harvested data tables (see {ref}`RN3_Generic_Harvesting_Data.md-tables-harvested-data`).
 
**Schema and names**  

The standard name for the template table schema is **[template]**.

Template table names should match the RN3 table names. 
If the dataflow, however, contains multiple tables with the same names, just in different datasets, and their data are harvested to the same Import database, the template table names need to be differentiated. The standard approach is to prefix the RN3 table name with an abbreviation of the specific RN3 dataset schema.

```{admonition} Example
:class: dropdown
The Habitatas directive reporting dataflow contains tables with the same names in its reporting dataset schemas. 
For example, the table *Maps* is present in both the *Reporting data - Habitats* and *Reporting data - Species* datasets. In the NatureArt17_Import database, the corresponding template tables are named *Habitats_Maps* and *Species_Maps*. 
For consistency, all template table names were prefixed with an abbreviation of their dataset schema.
```

Another, relatively simple option for handling tables with the same names is to use different table schemas for tables from different datasets.  
(There may be other options, like different dataset-specific import databases or data-collection-specific harvesting processes, but these may be unnecessarily complex, and we are not going to describe them.)

**Structure**

The template table structure should match that of the RN3 dataset table. The table columns must have the same names as the RN3 table fields (including the same letter case). 

In addition to the data columns, each template table must contain these 3 **metadata columns**:
- **[rn3_dataProviderCode]**
- **[rn3_snapshotId]**
- **[rn3_recordId]**
```{seealso}
See {ref}`RN3_Generic_Harvesting_Data.md-tables-templates-reference` for the details
```
The harvesting process will fail if any metadata column is missing.

The harvesting process will not fail if a template table contains misnamed data columns or is missing columns for any RN3 table field.  
The misnamed columns will not be populated.  
If any RN3 table fields should not, or need not, be harvested, excluding the respective column from the template table is a way to achieve that.

Each data column in the template table needs a **data type** appropriate for the values reported in the corresponding RN3 fields.

```{warning}
The harvesting process will fail if the template table column has an inappropriate data type. This includes text columns (nvarachar) with an insufficient character limit.  
To continue harvesting from the RN3 dataflow, correct the data type in both the affected template and the corresponding harvested data table, or disable harvesting for the specific release.
```

**Geometry data**  

By default, if an RN3 table contains geometry fields, and the corresponding template table contains columns with the same names (and an appropriate data type), the geometry values will be imported to these columns as GeoJSON strings.  
The harvesting process, however, can be configured to import geometry data differently. The data manager can decide to:  
- Convert the GeoJSON string to WKB and store it in a SQL Geometry column of the same or different name.  
- Keep the GeoJSON string and import it to an appropriate column of the same name, or drop it.  
- Store the converted geometry data from different RN3 geometry fields in different columns, or in the same column if every RN3 record will have geometry in just one of the fields.  

```{seealso}
See {ref}`RN3_Generic_Harvesting_Data.md-tables-geomfields` for details about the configuration of the geometry data harvesting.  
```

(RN3_Generic_Harvesting_Data.md-tables-templates-reference)=
#### Reference

Each template table must include the following metadata columns. It's recommended to add them to the front of the table.  

- **[rn3_dataProviderCode]** -  The code of the RN3 data provider (e.g., two-letter country code). Recommended data type: **nvarchar** of sufficient length.  
- **[rn3_snapshotId]** - The identifier of the release snapshot the data is part of. Recommended data type: **bigint**.  
- **[rn3_recordId]** - The unique identifier of the RN3 record. Recommended data type: **nvarchar(100)**.  

#### How to

(RN3_Generic_Harvesting_Data.md-tables-how-to-create-template-tables)=
##### Create [template] tables 

**SQL - example:**  

~~~~sql
USE [NatDA_Import]

--CREATE SCHEMA [template]

DROP TABLE IF EXISTS [NatDA_Import].[harvestedData].[ProtectedSite];
DROP TABLE IF EXISTS [NatDA_Import].[template].[ProtectedSite];
CREATE TABLE [NatDA_Import].[template].[ProtectedSite](
    [rn3_dataProviderCode] [nvarchar](5) NOT NULL,
    [rn3_snapshotId] [bigint] NOT NULL,
    [rn3_recordId] [nvarchar](100) NOT NULL,
    [inspireIDLocalId] [nvarchar](255) NULL,
    [inspireIDNamespace] [nvarchar](255) NULL,
    [inspireIDVersionId] [nvarchar](255) NULL,
    [legalFoundationDate] [nvarchar](50) NULL,
    [siteName] [nvarchar](255) NULL,
    [thematicIdentifier] [bigint] NULL,
    [Geometry_polygons] [geometry] NULL,
    [Geometry_points] [geometry] NULL,
    [sourceIdentifier] [nvarchar](255) NULL
)
~~~~


(RN3_Generic_Harvesting_Data.md-tables-harvested-data)=
### Harvested data tables

These Import database tables store the harvested data.  

**Schema and names**  

The standard name for the harvested data table schema is **[harvestedData]**.  

When the harvesting process runs for the first time for a specific dataflow, it creates the table schema and harvested data tables by copying the corresponding template tables. The harvested data tables will have the same names as the template tables.  

```{warning}
It's not recommended to create the harvested data tables manually.
```

**Structure**  

The harvested data tables' structure matches the structure of the corresponding template tables.

Any change made in a template table must be made in the corresponding harvested data table, if it already exists.  
You can also delete the harvested data table if it's empty or the data it already contains doesn't need to be preserved. The data harvesting process will create the table again next time it runs. 


(RN3_Generic_Harvesting_Data.md-tables-harvesting-jobs)=
### Harvesting jobs table

This Import database table contains a list of the harvesting jobs and their metadata.  

A harvesting job record is created for each viable release snapshot.  

The data harvesting process uses the table to identify which release snapshots have already been harvested, and adds new records when it detects new releases. At the end, it documents whether the specific release snapshot was successfully harvested, when it was harvested, or why it failed.  

**Schema and name**  

The standard table name is **[HarvestingJobs]** and the standard schema is **[metadata]**.  

When the harvesting process runs for the first time for a specific dataflow, it creates the table schema if it doesn't exist yet. It then also creates the table by copying the relevant content from the [RN3].[metadata].[HistoricalRelease] table. 

```{warning}
It's not recommended to create the harvesting jobs tables manually.
```

**Structure**  

The harvested data table structure matches the structure of the **[RN3].[metadata].[HistoricalRelease]** table, with a few **additional columns** (see {ref}`RN3_Generic_Harvesting_Data.md-tables-harvesting-jobs-reference` for the details).

```{seealso}
See **[HistoricalRelease]** table entry in the **Metadata tables {ref}`RN3_Generic_Harvesting_Metadata.md-tables-reference`** for more details.
```

(RN3_Generic_Harvesting_Data.md-tables-harvesting-jobs-reference)=
#### Reference

In addition to the columns from the [RN3].[metadata].[HistoricalRelease] table, the harvesting jobs table contains these additional columns:
- **[harvestDate]** - A timestamp when the release snapshot data was successfully harvested.
- **[jobId]** - An identifier of the data harvesting FME job that harvested the release snapshot.
- **[jobSummary]** - A summary of the harvesting job in JSON format.

(RN3_Generic_Harvesting_Data.md-tables-harvesting-jobs-explanation)=
#### Explanation

- At the beginning of the data harvesting, the process finds the release snapshots metadata in the [RN3].[metadata].[HistoricalRelease] that match the specific harvesting parameters and are not already in the harvesting jobs table. It then copies those with **[dcrelease] = 1** to the table. It also adds the **[jobId]** value to the record at that time.  
- The process then selects any harvesting jobs records with an empty **[harvestDate]** field, attempts to download the corresponding data from the data collection datasets, to process it and to import it into the harvested data tables.  
- The process adds the **[harvestDate]** value to the harvesting jobs record only if the harvesting was fully successful. This means it tries to harvest data from failed jobs next time it runs.  
- By manipulating the **[harvestDate]** value manually,  we can either prevent the process from harvesting specific release snapshots or force it to re-harvest them.
    ```{seealso}
    How to {ref}`RN3_Generic_Harvesting_Data.md-how-to-prevent-harvesting`.  
    How to {ref}`RN3_Generic_Harvesting_Data.md-how-to-reharvest-snapshot`.  
    ```
- The "status" in the **[jobSummary]** JSON will be "success" for successful harvesting jobs and "failure" for failed jobs. The "summary" will, for each table in the specific release snapshot, contain the final number of harvested records and other relevant numbers. Because a harvesting failure may be caused by a mismatch between the records entering a specific step and the number of records in the step's output, the summary numbers may help identify the concrete failure reason.

#### How to

(RN3_Generic_Harvesting_Data.md-how-to-prevent-harvesting)=
##### Prevent snapshot harvesting

When the harvesting process repeatedly fails to harvest a certain snapshot, and it can't be fixed otherwise, we may need to stop it from trying.

To prevent harvesting of a specific release snapshot that has a record in the harvesting jobs table, we manually add a [harvestDate] value to the record.  

We recommend adding a text value to the record's [jobSummary] field that explains why we're excluding the snapshot.

**SQL - example:**  

~~~~sql
UPDATE a
SET [harvestDate] = '2024-01-10 06:00:00'
    ,[jobSummary] = 'Harvesting manualy disabled. The snapshots contain no data, probably due to an RN3 platform error at release.'
--SELECT *
FROM [WISE_SOE_Import].[metadata].[HarvestingJobs] as a
WHERE [snapshotId] = 65849
~~~~


(RN3_Generic_Harvesting_Data.md-how-to-reharvest-snapshot)=
##### Reharvest snapshot data

To force the process to **reharvest** specific snapshots, we manually delete the [harvestDate] value from the corresponding records.  

**SQL - example:**  

~~~~sql
UPDATE a
SET [harvestDate] = NULL
--SELECT *
FROM [NatDA_Import].[metadata].[HarvestingJobs] as a
WHERE snapshotId IN ('105195', '105193', '105194')
~~~~

To reharvest just a single release snapshot, we can also run the process with the record's [snapshotId] value in the snapshotId user parameter. The [harvestDate] value will be automatically deleted at the start of the process.


(RN3_Generic_Harvesting_Data.md-tables-rn3-dataflow-metadata-tables)=
### RN3 dataflow metadata tables

The RN3 dataflow metadata tables are described in the {ref}`RN3_Generic_Harvesting_Metadata.md-tables` section of the RN3 Generic Harvesting - metadata document.

As mentioned previously, the tables must exist and be regularly updated for the data harvesting process to function.
 
The data harvesting process uses the following RN3 metadata tables:
- **[Dataflow]**
- **[HistoricRelease]**

The **[Dataflow]** table provides the RN3 dataflow ApiKey value the process uses to authorise the data download request.

The **[HistoricRelease]** table is the source of the information on release snapshots that need to be harvested.


(RN3_Generic_Harvesting_Data.md-tables-dataflow-tables)=
### Dataflow tables table

This RN3 database table contains a list of **RN3 dataflow tables** to be harvested, parameters of the corresponding **template** and **harvested data** tables, and other related parameters.

**Schema and name**  

The standard table name is **[DataflowTables]** and the standard schema is **[metadata]**.  

**Structure**  

The [RN3].[metadata].[DataflowTables] table contains columns for five parameter groups:  

1. **Dataflow-related** parameters  
	- **[obligationId]**  
	- **[dataflowId]**  
	- **[dataflowName]**  
	- **[dataCollectionId]**  

2. **RN3 table** parameters
	- **[table_RN3_Name]**

3. The **harvested data table** parameters  
	- **[table_SQL_data_DatabaseConnection]**  
	- **[table_SQL_data_Database]**  
	- **[table_SQL_data_Schema]**  
	- **[table_SQL_data_Name]**  

4. The **template table** parameters  
	- **[table_SQL_template_DatabaseConnection]**  
	- **[table_SQL_template_Database]**  
	- **[table_SQL_template_Schema]**  
	- **[table_SQL_template_Name]**  

5. The **geometry data** related parameters  
	- **[flag_keep_geojson]**  
	- **[flag_update_geometry]**  


(RN3_Generic_Harvesting_Data.md-tables-dataflow-tables-reference)=
#### Reference

text

(RN3_Generic_Harvesting_Data.md-tables-dataflow-tables-explanation)=
#### Explanation

- If the [RN3].[metadata].[DataflowTables] table doesn't exist yet, a responsible data manager must create it.
	```{seealso}
	See How to {ref}`RN3_Generic_Harvesting_Data.md-tables-how-to-create-dataflow-tables-table` for an example.
	```
- The data manager needs to add a record for each RN3 table they want to harvest data from and provide all mandatory values.
	```{seealso}
	See How to {ref}`RN3_Generic_Harvesting_Data.md-tables-how-to-populate-dataflow-tables-table` for an example.
	```

```{warning}
Be careful not to delete the [RN3].[metadata].[DataflowTables] table if it already exists, or delete or change records for other dataflows that may be using the same table. 
```



#### How to

(RN3_Generic_Harvesting_Data.md-tables-how-to-create-dataflow-tables-table)=
##### Create [metadata].[DataflowTables] table

**SQL - example:**  


~~~~sql
DROP TABLE [RN3].[metadata].[DataflowTables]

CREATE TABLE [RN3].[metadata].[DataflowTables](
    [obligationId] [int] NOT NULL,
    [dataflowId] [bigint] NOT NULL,
    [dataflowName] [nvarchar](255) NULL,
    [dataCollectionId] [bigint] NOT NULL,
    [table_RN3_Name] [nvarchar] (255) NOT NULL,
    [table_SQL_data_DatabaseConnection] [nvarchar](255) NOT NULL,
    [table_SQL_data_Database] [nvarchar](255) NOT NULL,
    [table_SQL_data_Schema] [nvarchar](255) NOT NULL,
    [table_SQL_data_Name] [nvarchar](255) NOT NULL,
    [table_SQL_template_DatabaseConnection] [nvarchar](255) NOT NULL,
    [table_SQL_template_Database] [nvarchar](255) NOT NULL,
    [table_SQL_template_Schema] [nvarchar](255) NOT NULL,
    [table_SQL_template_Name] [nvarchar](255) NOT NULL,
    [flag_keep_geojson] [bit] NOT NULL,
    [flag_update_geometry] [bit] NOT NULL,

PRIMARY KEY CLUSTERED 
(
    [dataCollectionId] ASC,
    [table_RN3_Name] ASC
) WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON) ON [PRIMARY]
) ON [PRIMARY]
~~~~

(RN3_Generic_Harvesting_Data.md-tables-how-to-populate-dataflow-tables-table)=
##### Populate [metadata].[DataflowTables] table

**SQL - example:**  

~~~~sql
--DELETE
--SELECT *
--FROM [RN3].[metadata].[DataflowTables]
--WHERE [dataflowId] IN (<dataflowId>,<dataflowId>)

INSERT INTO [RN3].[metadata].[DataflowTables]
(
    [obligationId]
    ,[dataflowId]
    ,[dataflowName]
    ,[dataCollectionId]
    ,[table_RN3_Name]
    ,[table_SQL_data_DatabaseConnection]
    ,[table_SQL_data_Database]
    ,[table_SQL_data_Schema]
    ,[table_SQL_data_Name]
    ,[table_SQL_template_DatabaseConnection]
    ,[table_SQL_template_Database]
    ,[table_SQL_template_Schema]
    ,[table_SQL_template_Name]
    ,[flag_keep_geojson]
    ,[flag_update_geometry]
)

VALUES
-- one record for each dataflow table that should be harvested
    ( <obligationId>,<dataflowId>,'<dataflowName>',<dataCollectionId>,'<rn3_tablename>','<Import_database_FME_connection_name>','<Import_database_name','harvestedData','<sql_data_table_name>','<Import_database_FME_connection_name>','<Import_database_name','template','<sql_template_table_name>',0,0 )
    ,( <obligationId>,<dataflowId>,'<dataflowName>',<dataCollectionId>,'<rn3_tablename>','<Import_database_FME_connection_name>','<Import_database_name','harvestedData','<sql_data_table_name>','<Import_database_FME_connection_name>','<Import_database_name','template','<sql_template_table_name>',0,1 )
~~~~


(RN3_Generic_Harvesting_Data.md-tables-geomfields)=
### Geometry fields table

text

(RN3_Generic_Harvesting_Data.md-tables-geomfields-reference)=
#### Reference

text

(RN3_Generic_Harvesting_Data.md-tables-geomfields-explanation)=
#### Explanation

text

#### How to

(RN3_Generic_Harvesting_Data.md-tables-tables-how-to-create-geomfields-table)=
##### Create [metadata].[DataflowTables_GeometryFields] table

**SQL - example:**  


~~~~sql
Select * from table

~~~~


(RN3_Generic_Harvesting_Data.md-tables-harvesting-parameters)=
### Harvesting parameters table

text


(RN3_Generic_Harvesting_Data.md-tables-harvesting-parameters-reference)=
#### Reference

text


(RN3_Generic_Harvesting_Data.md-tables-harvesting-parameters-explanation)=
#### Explanation

text

#### How to

(RN3_Generic_Harvesting_Data.md-tables-tables-how-to-create-harvesting-parameters-table)=
##### Create [metadata].[HarvestingParameters] table

**SQL - example:**  


~~~~sql
Select * from table

~~~~






(RN3_Generic_Harvesting_Data.md-fme-workspace)=
## FME workspace 

The metadata harvesting FME workspace does the following: 
- text
- text

**Latest version:**  
<https://fme.discomap.eea.europa.eu/fmeserver/workspaces/run/Dataflows_RN3_Generic_Processes/RN3_Generic_Harvesting_Data_v3a.fmw>

(RN3_Generic_Harvesting_Data.md-fme-workspace-user-parameters)=
### User parameters

#### Reference

**Mandatory parameters:**  
- **p1**
- **p2**

**Optional parameters:**  
- **p3**
- **p4**

#### Explanation

- text
- The FME connection referred to in the **RN3_metadata_databaseConnection** parameter must exist in the EEA's FME Flow server. The connection must be non-JDBC. The FME Flow server needs *db_ddladmin* or *db_owner* permissions on the linked database to access it and write to the metadata tables.  

    ```{seealso}
	How to {ref}`RN3_Generic_Harvesting_Data.md-how-to-create-database-connection`  
	How to {ref}`RN3_Generic_Harvesting_Data.md-how-to-add-fme-database-user`  
	``` 

(RN3_Generic_Harvesting_Data.md-fme-workspace-schedule)=
### Schedule

The data harvesting processes are commonly run on a regular schedule. This can be achieved by creating an FME Schedule (or Automation) that triggers the FME workspace at specified intervals and supplies it with corresponding user parameters.  
The most commonly used frequency for RN3 metadata harvesting is once a day. It's highly recommended to run it late at night.  
The generic metadata harvesting should run about 10-15 minutes before the data harvesting so it has time to finish.

```{seealso}
How to {ref}`RN3_Generic_Harvesting_Data.md-how-to-create-fme-schedule`  
``` 

### How to

(RN3_Generic_Harvesting_Data.md-how-to-create-database-connection)=
#### Create a database connection on FME Flow

- Follow <https://support.safe.com/hc/en-us/articles/25407463461517-FAQ-Database-Connections-on-FME-Flow>.
- If you don't have permissions to add a connection, or are not confident enough to create it, ask EEA Service Desk to create it for you.

(RN3_Generic_Harvesting_Data.md-how-to-add-fme-database-user)=
#### Add FME Flow server as a user to an MS SQL database

- Follow <https://learn.microsoft.com/en-us/sql/relational-databases/security/authentication-access/create-a-database-user?view=sql-server-ver17#create-a-user-with-ssms>.
- Add a user with eeadmz1\fmeservice as the user name and login name.
- Select appropriate permission options on the Membership page.
- If you don't have permissions to add users to the database, ask EEA Service Desk to add it.

(RN3_Generic_Harvesting_Data.md-how-to-create-fme-schedule)=
#### Create a Schedule on FME Flow.
- Follow <https://docs.safe.com/fme/html/FME-Flow/WebUI/schedules.htm>.
- If you don't have permissions to create a Schedule, or are not confident enough to create it, ask the EEA Service Desk or a colleague with appropriate permissions and experience to create it for you.
