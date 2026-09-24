(rn3-generic-update-reference)=
# RN3 Generic Update - Reference datasets

<hr class="double">

**Complexity:**  
<span class="stars">★★★★☆</span>


## Introduction

Reference dataset update is an FME-orchestrated process that updates selected RN3 dataflow **reference datset tables** if the corresponding **reference data source** has been updated since the last update. The reference data sources are tables or views in an **MS SQL database** (or databases).

The process can be triggered manually or, more commonly, scheduled to run regularly.  

<hr class="thick">

## Quick setup

1. Ensure the **metadata harvesting process** is set up for the dataflows you want to harvest data from, and the **RN3 metadata tables** exist.  
     - See {ref}`RN3_Generic_Update_Reference.md-tables-rn3-dataflow-metadata-tables`.  
2. Ask the EEA Service Desk to create the **Import database** if it doesn't exist yet, and if you don't want to use a different database.  
    - See {ref}`RN3_Generic_Update_Reference.md-databases-import` database for more details.  
3. Create **Template tables** in the Import database, one for each RN3 table you want to harvest.  
    - See How to {ref}`RN3_Generic_Update_Reference.md-tables-how-to-create-template-tables` for an SQL example.  
    - See {ref}`RN3_Generic_Update_Reference.md-tables-templates` for more details.  
4. Create the **Dataflow Tables table** in the RN3 database if it doesn't exist yet, and insert one record for each RN3 table you want to harvest.  
    - See How to {ref}`RN3_Generic_Update_Reference.md-tables-how-to-create-dataflow-tables-table` for an SQL example.  
    - See How to {ref}`RN3_Generic_Update_Reference.md-tables-how-to-populate-dataflow-tables-table` for an SQL example.  
    - See {ref}`RN3_Generic_Update_Reference.md-tables-dataflow-tables` for more details.  
5. If you want to harvest geometry data from one or more of the RN3 tables, create the **Geometry Fields table** in the RN3 database if it doesn't exist yet. Insert one record for each combination of the source RN3 geometry field and the target SQL geometry field.  
    - See How to {ref}`RN3_Generic_Update_Reference.md-tables-tables-how-to-create-geomfields-table` for an SQL example.  
    - See How to {ref}`RN3_Generic_Update_Reference.md-tables-tables-how-to-populate-geomfields-table` for an SQL example.  
    - See {ref}`RN3_Generic_Update_Reference.md-tables-geomfields` for more details.  
6. Create the **Harvesting Parameters table** in the RN3 database if it doesn't exist yet, and insert one record for each RN3 data collection you want to harvest.  
    - See How to {ref}`RN3_Generic_Update_Reference.md-tables-how-to-create-harvesting-parameters-table` for an SQL example.  
    - See How to {ref}`RN3_Generic_Update_Reference.md-tables-how-to-populate-harvesting-parameters-table` for an SQL example.  
    - See {ref}`RN3_Generic_Update_Reference.md-tables-harvesting-parameters` for more details.  
7. **Create a data harvesting schedule** on the EEA's FME Flow server.  
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

The tables do not need to be located on the same MS SQL Server, but it's recommended they do, as it makes working with them easier.  

<hr class="thick">


## Databases

The data harvesting setup is flexible regarding the names and number of databases. The standard setup model, however, works with two databases.  
- **Source** database  
- **RN3** database  

*In this documentation, we will be using this model and the database names.*  

(RN3_Generic_Update_Reference.md-databases-source)=
### Source

This is a dataflow-specific database.  
In a standard EEA dataflow database model, the role of the Source database is usually taken by the Production database, but depending on the actual source of the reference data it may also be the Import database, the 'non-suffix' database or any other database.  
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

There is no standard convention for naming the Source tables. The data manager should use names and schemas that best fit their way of reference data management and followed conventions.  
It's howevere recommended that the name helps to match the source table with the target RN3 reference dataset table.

It's also up to data manager's decision if the reference source data is provided in a dedicated Table, or if it's 'produced' on the fly by a View.

<ins>**Structure**</ins>

The Source table structure should match that of the target **RN3 reference dataset table**. The table columns must have the same names as the RN3 table fields (including the same letter case).  

In addition to the data columns, each Source table must contain a *datetime* column including the timestamp of when the specific record was created or updated last time.  
This column is used to identify if the reference data source has changed since the last time the corresponding RN3 reference dataset table has been updated.


(RN3_Generic_Update_Reference.md-tables-source-tables-reference)=
#### Reference

Each Source table or view must include a datetime column to include the timestamp the last update of the reference record.
The data manager is free to choose the name of this column, but we will be using name **lastUpdate** in the further text.


(RN3_Generic_Update_Reference.md-tables-source-tables-explanation)=
#### Explanation

- The individual **Source tables or views** should be created in cooperation with the **responsible dataflow owner** (e.g., data steward).
- The update process will fail if a Source table or view misses the **lastUpdate column**, or if it is empty for all records.  
- The update process will not fail if a Source table or view contains misnamed data columns or is missing columns for any RN3 table field. The misnamed columns will not be used and RN3 table fields with missing Source columns will be empty after the update.  
- Each data column in the Source table or view should contain only data that are appropriate for the corresponding RN3 fields. The update process will not fail if the data type of the Source data is incorrect, but it will most likely cause validation issues in the RN3 dataflow.  
	
	```{important}
	If the *RN3 reference dataset table* contains any **geometry fields**, the corresponding Source table or view needs to contain the geometry values formatted as **Extended GeoJSON** string, not as SQL geometry!  
	```
- The update compares the highest timestamp value in the **lastUpdate** column of the Source table, with the highest **[tableUpdated]** value of the corresponding **Update Jobs table** records. 
    - If the *lastUpdate* value is higher, the process selects all data from the Source table and uses it to update the corresponding RN3 reference dataset table. 
	- If the *lastUpdate* value is lower, the process will not update the corresponding RN3 reference dataset table. 
	- If no Source table has a higher *lastUpdate* value, the update process ends there.

<hr>

(RN3_Generic_Update_Reference.md-tables-update-jobs)=
### Update Jobs table

In the standard model, the Update Jobs table is placed in the **Source database**.  

The table contains a list of the update jobs and their metadata.  

The process uses the table to identify the latest update for the corresponding RN3 reference dataset table. At the end of the update it adds a new records for each reference table it updated, or attempted to update, and documents the job result.  

<ins>**Schema and name**</ins>  

The standard table name is **[ReferenceUpdateJobs]** and the standard schema is **[metadata]**.  

When the update process runs for a specific dataflow for the first time, it creates the schema and the table if they don't exist yet.  

```{warning}
It's not recommended to create the Update Jobs tables manually.  
```

<ins>**Structure**</ins>  

The Update Jobs table structure contains metadata columns necessary to uniquely identify each job at the reference table level, as well as the result of each job.  


(RN3_Generic_Update_Reference.md-tables-update-jobs-reference)=
#### Reference

In addition to the columns from the [RN3].[metadata].[HistoricalRelease] table, the Harvesting Jobs table contains these additional columns:  
- **[dataflowId]** - The RN3 dataflow identifier.  
- **[datasetId]** - The RN3 reference datset identifier.  
- **[datasetName]** - The RN3 reference dataset name.  
- **[tableSchemaId]** - The RN3 reference dataset table schema identifier.  
- **[tableName]** - The RN3 reference dataset table name.  
- **[tableUpdated]** - A timestamp when the reference table was successfully updated.  
- **[jobId]** - An identifier of the FME job that run the reference update process.  
- **[jobExecuted]** - A timestamp of when the FME job started. 
- **[jobSummary]** - A summary of the reference update job.  

(RN3_Generic_Update_Reference.md-tables-update-jobs-explanation)=
#### Explanation

- At the beginning of the data harvesting, the process finds the release snapshots metadata in the **[RN3].[metadata].[HistoricalRelease]** that match the specific harvesting parameters and are not already in the Harvesting Jobs table. It then copies those with **[dcrelease] = 1** to the table. It also adds the **[jobId]** value to the record at that time.  
- The process then selects any harvesting jobs records with an empty **[harvestDate]** field and attempts to download the corresponding data from the data collection datasets. It then tries to process it and import it into the Harvested Data tables.  
- The process adds the **[harvestDate]** value to the harvesting jobs record only if the harvesting was fully successful. This means it tries to harvest data from failed jobs next time it runs.  
- By manipulating the **[harvestDate]** value manually, we can either prevent the process from harvesting specific release snapshots or force it to re-harvest them.
    ```{seealso}
    How to {ref}`RN3_Generic_Update_Reference.md-how-to-prevent-harvesting`.  
    How to {ref}`RN3_Generic_Update_Reference.md-how-to-reharvest-snapshot`.  
    ```
- The "status" in the **[jobSummary]** JSON will be "success" for successful harvesting jobs and "failure" for failed jobs. The "summary" will, for each table in the specific release snapshot, contain the final number of harvested records and other relevant numbers. Because a harvesting failure may be caused by a mismatch between the records entering a specific processing step and the number of records in the step's output, the summary numbers may help identify the concrete failure reason.  

#### How to

(RN3_Generic_Update_Reference.md-how-to-prevent-harvesting)=
##### Prevent snapshot harvesting

When the harvesting process repeatedly fails to harvest a certain snapshot, and it can't be fixed otherwise, we may need to stop it from trying.  

To prevent harvesting of a specific release snapshot that has a record in the Harvesting Jobs table, we manually add a [harvestDate] value to the record.  

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


(RN3_Generic_Update_Reference.md-how-to-reharvest-snapshot)=
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
This method is mostly viable only when we execute the harvesting process manually, e.g., during testing.  

<hr>

(RN3_Generic_Update_Reference.md-tables-rn3-dataflow-metadata-tables)=
### RN3 Dataflow Metadata tables

The RN3 Dataflow Metadata tables are described in the **{ref}`RN3_Generic_Harvesting_Metadata.md-tables`** section of the **RN3 Generic Harvesting - metadata** document.  

As mentioned previously, the tables must exist and be regularly updated for the data harvesting process to function.  
 
The data harvesting process uses the following RN3 metadata tables:  
- **[Dataflow]**  
- **[HistoricRelease]**  

The **[Dataflow]** table provides the RN3 dataflow ApiKey value the process uses to authorise the data download request.  

The **[HistoricRelease]** table is the source of the information on release snapshots that need to be harvested, as explained in the {ref}`RN3_Generic_Update_Reference.md-tables-update-jobs` part.  

<hr>

(RN3_Generic_Update_Reference.md-tables-dataflow-tables)=
### Dataflow Tables table

In the standard model, the Dataflow Tables table is placed in the **RN3 database**.  

It contains a list of **RN3 dataflow tables** to be harvested, parameters of the corresponding **template** and **harvested data** tables, and other relevant parameters.  

<ins>**Schema and name**</ins>  

The standard table name is **[DataflowTables]** and the standard schema is **[metadata]**.  

<ins>**Structure**</ins>  

The [RN3].[metadata].[DataflowTables] table contains columns for five parameter groups:  

1. **Dataflow-related** parameters  
	- [obligationId], [dataflowId], [dataflowName], [dataCollectionId]  

2. **RN3 table** parameters
	- [table_RN3_Name]

3. **Harvested Data table** parameters  
	- [table_SQL_data_DatabaseConnection], [table_SQL_data_Database], [table_SQL_data_Schema], [table_SQL_data_Name]  

4. **Template table** parameters  
	- [table_SQL_template_DatabaseConnection], [table_SQL_template_Database], [table_SQL_template_Schema], [table_SQL_template_Name]  

5. The **geometry data** related parameters  
	- [flag_keep_geojson], [flag_update_geometry], [SQL_geometry_reproject_srid]


(RN3_Generic_Update_Reference.md-tables-dataflow-tables-reference)=
#### Reference

[RN3].[metadata].[DataflowTables] table columns:  
- **[obligationId]**  - The identifier of the reporting obligation assigned to the RN3 dataflow. It must match the [obligationId] in the [RN3].[metadata].[Dataflow].  
- **[dataflowId]**  - The RN3 dataflow identifier.  
- **[dataflowName]**  - Optional. The name of the RN3 dataflow.
- **[dataCollectionId]**  - The RN3 data collection identifier.  
- **[table_RN3_Name]** - The name of the RN3 table.  
- **[table_SQL_data_DatabaseConnection]**  - Name of the FME Database connection linked to the MS SQL database containing the Harvested Data table.  
- **[table_SQL_data_Database]**  -  Name of the MS SQL database containing the Harvested Data table.  
- **[table_SQL_data_Schema]**  - Name of the Harvested Data table schema (standard is 'harvestedData').  
- **[table_SQL_data_Name]**  - Name of the Harvested Data table to store data from the RN3 table.  
- **[table_SQL_template_DatabaseConnection]**  - Name of the FME Database connection linked to the MS SQL database containing the Template table.  
- **[table_SQL_template_Database]** - Name of the MS SQL database containing the Template table.    
- **[table_SQL_template_Schema]** - Name of the Template table schema (standard is 'template').  
- **[table_SQL_template_Name]** - Name of the Template table that is the source for the Harvested Data table.  
- **[flag_keep_geojson]** - Optional. A boolean value indicates whether the GeoJSON, harvested from geometry field(s) in the RN3 table, should be stored in the field(s) of the same name in the SQL table (1 = yes, 0 = no).
- **[flag_update_geometry]**  - Optional. A boolean value indicates whether the Harvested Data table geometry field should be updated using the data extracted from the corresponding RN3 geometry field (1 = yes, 0 = no).
- **[SQL_geometry_reproject_srid]** - Optional. Integer part of the EPSG notation of the specific CRS to which the SQL geometry should be reprojected (e.g., 4326).


(RN3_Generic_Update_Reference.md-tables-dataflow-tables-explanation)=
#### Explanation

- If the [RN3].[metadata].[DataflowTables] table doesn't exist yet, a responsible data manager must create it.
	```{seealso}
	See How to {ref}`RN3_Generic_Update_Reference.md-tables-how-to-create-dataflow-tables-table` for an example.
	```
- The data manager needs to add a record for each RN3 table they want to harvest data from and provide all mandatory values.
	```{seealso}
	See How to {ref}`RN3_Generic_Update_Reference.md-tables-how-to-populate-dataflow-tables-table` for an example.
	```
- It is in this table that the data manager specifies whether the template and Harvested Data tables are located in different databases or have non-standard schemas and names. As mentioned before, the Template tables can even be in a database located on a different MS SQL server than the other tables.

```{warning}
Be careful not to delete the [RN3].[metadata].[DataflowTables] table if it already exists, or delete or change records for other dataflows that use the same table.  
```

- The FME database connections referred to in the **table_SQL_data_DatabaseConnection** and **table_SQL_template_DatabaseConnection** parameters must exist in the EEA's FME Flow server. The connections must be JDBC, and their name should end with the suffix '_JDBC'. The FME Flow server needs *db_ddladmin* or *db_owner* permissions on the linked databases to access them and write to the tables.  

    ```{seealso}
	How to {ref}`RN3_Generic_Update_Reference.md-how-to-create-database-connection`  
	How to {ref}`RN3_Generic_Update_Reference.md-how-to-add-fme-database-user`  
	``` 

- The database connection parameters may contain both non-JDBC and JDBC versions of the connection name. The non-JDBC connection names must match the JDBC one, just without the '_JDBC' suffix. The data harvesting process automatically creates attributes with the JDBC versions of the database connections if the respective parameters contain non-JDBC ones.


-  The **[flag_keep_geojson]**, **[flag_update_geometry]**, **[SQL_geometry_reproject_srid]** values must be NULL for RN3 tables without geometry fields. 

-  The **[flag_keep_geojson]** value can be 1 only if the data is to be downloaded from RN3 as *JSON* or *CSV* files. Only in these formats is the geometry present as GeoJSON. If the download format is parquet, it contains geometry data in extended WKB format. There is no valid reason for trying to store it in the database in that format. The harvesting process also won't convert WKB to GeoJSON.
 	```{seealso}
	See **etlExportVersion** in FME workspace {ref}`RN3_Generic_Update_Reference.md-fme-workspace-user-parameters` for how to decide and specify the download file format.
	```

- If the **[flag_update_geometry]** is 1, the Geometry Fields table must exist and contain information about the relation between the geometry fields in the RN3 and the Harvested Data table.
	```{seealso}
	See {ref}`RN3_Generic_Update_Reference.md-tables-geomfields`
	```

- The **[SQL_geometry_reproject_srid]** must be left empty if the geometry should be stored in the Harvested Data table in its original CRS. 
- If the **[flag_update_geometry]** = 1, the harvesting process automatically creates an auxiliary table **[NatDA_Import].[harvestedData].[auxGeom]**. It uses it as temporary storage for geometry before inserting it into the Harvested Data table. You can ignore the table, or safely delete it once the data harvesting period has ended.

#### How to

(RN3_Generic_Update_Reference.md-tables-how-to-create-dataflow-tables-table)=
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
    [SQL_geometry_reproject_srid] [int] NULL

PRIMARY KEY CLUSTERED 
(
    [dataCollectionId] ASC,
    [table_RN3_Name] ASC
) WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON) ON [PRIMARY]
) ON [PRIMARY]
~~~~

(RN3_Generic_Update_Reference.md-tables-how-to-populate-dataflow-tables-table)=
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
    ,[SQL_geometry_reproject_srid]
)

VALUES
-- one record for each dataflow table that should be harvested
    ( <obligationId>,<dataflowId>,'<dataflowName>',<dataCollectionId>,'<rn3_tablename>','<Import_database_FME_connection_name>','<Import_database_name','harvestedData','<sql_data_table_name>','<Import_database_FME_connection_name>','<Import_database_name','template','<sql_template_table_name>',0,0,NULL )
    ,( <obligationId>,<dataflowId>,'<dataflowName>',<dataCollectionId>,'<rn3_tablename>','<Import_database_FME_connection_name>','<Import_database_name','harvestedData','<sql_data_table_name>','<Import_database_FME_connection_name>','<Import_database_name','template','<sql_template_table_name>',0,1,NULL )
~~~~

<hr>

(RN3_Generic_Update_Reference.md-tables-geomfields)=
### Geometry Fields table

The Geometry Fields table must be placed in the same database as the **{ref}`RN3_Generic_Update_Reference.md-tables-dataflow-tables`**.

If the table doesn't exist yet, the data manager must create it if they want to harvest geometry data from the geometry fields in the dataflow.

The table is basically an extension of the Dataflow Tables table.

<ins>**Schema and name**</ins>  

The table must have the name of the Dataflow Tables table suffixed with '_GeometryFields'. It must use the same table schema.  
So, in the standard model, it is **[metadata].[DataflowTables_GeometryFields]**.   

<ins>**Structure**</ins>  

The table contains selected columns from the Dataflow Tables table and these additional fields:
- **[field_RN3_Geometry]**
- **[field_SQL_Geometry]**

(RN3_Generic_Update_Reference.md-tables-geomfields-reference)=
#### Reference

The [metadata].[DataflowTables_GeometryFields] table columns:  
- **[dataflowId]** - Optional. The RN3 dataflow identifier.  
- **[dataflowName]** - Optional. The name of the RN3 dataflow.  
- **[dataCollectionId]**  - The RN3 data collection identifier.  
- **[table_RN3_Name]** - The name of the RN3 table.  
- **[field_RN3_Geometry]** - The name of the RN3 table geometry field.
- **[field_SQL_Geometry]** - The name of the geometry field in the Harvested Data table, where the RN3 field geometry should be stored.

(RN3_Generic_Update_Reference.md-tables-geomfields-explanation)=
#### Explanation

- The **[field_RN3_Geometry]** and **[field_SQL_Geometry]** can have the same or different value.
- The fields **[field_RN3_Geometry]** and **[field_SQL_Geometry]** must be different if we want to convert and store the geometry data as SQL geometry and also keep it as a GeoJSON value, which we indicate by setting the **[flag_keep_geojson] = 1** and **[flag_update_geometry] = 1** in the {ref}`RN3_Generic_Update_Reference.md-tables-dataflow-tables`. In this situation, the corresponding **Template table** must contain two 'geometry' columns: one *[nvarchar]\(max)* column for the GeoJSON value and a *[geometry]* type column for the actual geometry. The column that stores the GeoJSON value must have the same name as the RN3 field.
- If the RN3 table contains multiple different geometry fields (e.g., one for point and the other for polygon geometries), but only one of the fields can have a value in the released data, it's possible to store geometries from all RN3 fields in just one SQL field. In that case, add one record for each RN3 field to the Geometry Fields table, and give them the same **[field_SQL_Geometry]** value.

#### How to

(RN3_Generic_Update_Reference.md-tables-tables-how-to-create-geomfields-table)=
##### Create [metadata].[DataflowTables_GeometryFields] table

**SQL - example:**  

~~~~sql
CREATE TABLE [RN3].[metadata].[DataflowTables_GeometryFields](
	[dataflowId] [bigint] NULL,
	[dataflowName] [nvarchar](255) NULL,
	[dataCollectionId] [bigint] NOT NULL,
	[table_RN3_Name] [nvarchar](255) NOT NULL,
	[field_RN3_Geometry] [nvarchar](255) NOT NULL,
	[field_SQL_Geometry] [nvarchar](255) NOT NULL
) ON [PRIMARY]

ALTER TABLE [RN3].[metadata].[DataflowTables_GeometryFields]  WITH CHECK ADD  CONSTRAINT [FK_DataflowTables_GeometryFields_DataflowTables] FOREIGN KEY([dataCollectionId], [table_RN3_Name])
REFERENCES [RN3].[metadata].[DataflowTables] ([dataCollectionId], [table_RN3_Name])
ON DELETE CASCADE
~~~~

(RN3_Generic_Update_Reference.md-tables-tables-how-to-populate-geomfields-table)=
##### Populate [metadata].[DataflowTables_GeometryFields] table

**SQL - example:**  

~~~~sql
INSERT INTO [RN3].[metadata].[DataflowTables_GeometryFields]
    ([dataflowId]
    ,[dataflowName]
    ,[dataCollectionId]
    ,[table_RN3_Name]
    ,[field_RN3_Geometry]
    ,[field_SQL_Geometry])
VALUES
	-- one record for each combination of RN3 geometry field and SQL geometry field
	-- if RN3 table has multiple geometry fields, but one RN3 record can have geometry reported only in one of them, both RN3 fields can be mapped to the same SQL field in the corresponding SQL table
	(<dataflowId>,'<dataflowName>',<dataCollectionId>,'<rn3_tableName>','<rn3_geometry_fieldName>','sql_geometry_fieldName')
	,(<dataflowId>,'<dataflowName>',<dataCollectionId>,'<rn3_tableName>','<rn3_geometry_fieldName>','sql_geometry_fieldName')

~~~~

<hr>

(RN3_Generic_Update_Reference.md-tables-harvesting-parameters)=
### Harvesting Parameters table

In the standard model, the Harvesting Parameters table is placed in the **RN3 database**.  

It contains the core parameters used by the data harvesting process, one record for each RN3 data collection we want to harvest. 
The majority of the parameters identify the other metadata tables (Harvesting Jobs, RN3 metadata, Dataflow tables) the harvesting process must use to harvest data from the given RN3 dataflow and its data collections.

 <ins>**Schema and name**</ins>  

The standard table name is **[HarvestingParameters]** and the standard schema is **[metadata]**.  

<ins>**Structure**</ins>  

The [RN3].[metadata].[HarvestingParameters] table contains columns for five parameter groups:  

1. **Dataflow-related** parameters  
	- [obligationId], [dataflowId], [dataflowName], [dataCollectionId]  

2. **RN3 metadata tables** parameters
	- [metadata_DatabaseConnection], [metadata_Database], [metadata_Schema], [metadata_Table_Dataflow], [metadata_Table_DataCollection], [metadata_Table_HistoricRelease], [metadata_Table_ReportingDataset]

3. **Dataflow Tables table** parameters  
	- [dataflowTables_DatabaseConnection], [dataflowTables_Database], [dataflowTables_Schema], [dataflowTables_Table]

4. **Harvesting Jobs table** parameters  
	- [harvestingJobs_DatabaseConnection], [harvestingJobs_Database], [harvestingJobs_Schema], [harvestingJobs_Table] 

5. The **attachment** related parameters  
	- [attachmentParentPath]


(RN3_Generic_Update_Reference.md-tables-harvesting-parameters-reference)=
#### Reference

- **[obligationId]**  - The identifier of the reporting obligation assigned to the RN3 dataflow. It must match the [obligationId] in the [RN3].[metadata].[Dataflow].  
- **[dataflowId]**  - The RN3 dataflow identifier.  
- **[dataflowName]**  - Optional. The name of the RN3 dataflow.  
- **[dataCollectionId]**  - The RN3 data collection identifier.  
- **[metadata_DatabaseConnection]** - Name of the FME Database connection linked to the MS SQL database containing the RN3 metadata tables.  
- **[metadata_Database]** - Name of the MS SQL database containing the RN3 metadata tables (standard is 'RN3').  
- **[metadata_Schema]** - Name of the RN3 metadata tables schema (standard is 'metadata').  
- **[metadata_Table_Dataflow]** - Name of the table containing RN3 dataflow metadata (standard is 'Dataflow').  
- **[metadata_Table_DataCollection]** - Name of the table containing RN3 data collection metadata (standard is 'DataCollection').  
- **[metadata_Table_HistoricRelease]** - Name of the table containing RN3 release snapshot metadata (standard is 'HistoricRelease').  
- **[metadata_Table_ReportingDataset]** - Name of the table containing RN3 reporting dataset metadata (standard is 'ReportingDataset').  
- **[dataflowTables_DatabaseConnection]** - Name of the FME Database connection linked to the MS SQL database containing the Dataflow Tables table.  
- **[dataflowTables_Database]** - Name of the MS SQL database containing the Dataflow Tables table (standard is 'RN3').  
- **[dataflowTables_Schema]** - Name of the Dataflow Tables table schema (standard is 'metadata'). 
- **[dataflowTables_Table]** - Name of the Dataflow Tables table (standard is 'DataflowTables').
- **[harvestingJobs_DatabaseConnection]** - Name of the FME Database connection linked to the MS SQL database containing the Harvesting Jobs table.  
- **[harvestingJobs_Database]** - Name of the MS SQL database containing the Harvesting Jobs table (standardly it is the dataflow-specific Import table).  
- **[harvestingJobs_Schema]** - Name of the Harvesting Jobs table schema (standard is 'metadata'). 
- **[harvestingJobs_Table]** - Name of the Harvesting Jobs table (standard is 'HarvestingJobs').
- **[attachmentParentPath]** - Optional. Full path to the parent directory where attachment files are stored.  


(RN3_Generic_Update_Reference.md-tables-harvesting-parameters-explanation)=
#### Explanation

- If the [RN3].[metadata].[HarvestingParameters] table doesn't exist yet, a responsible data manager must create it.
	```{seealso}
	See How to {ref}`RN3_Generic_Update_Reference.md-tables-how-to-create-harvesting-parameters-table` for an example.
	```
- The data manager needs to add a record for each RN3 data collection they want to harvest data from and provide all mandatory values.
	```{seealso}
	See How to {ref}`RN3_Generic_Update_Reference.md-tables-how-to-populate-harvesting-parameters-table` for an example.
	```
- It is in this table that we specify the databases, table schemas and names of metadata tables, whether they follow the standard or differ from it.  

```{warning}
Be careful not to delete the [RN3].[metadata].[HarvestingParameters] table if it already exists, or delete or change records for other dataflows that use the same table.  
```

- The FME database connections referred to in the **metadata_DatabaseConnection**, **dataflowTables_DatabaseConnection** and **table_SQL_template_DatabaseConnection** parameters must exist in the EEA's FME Flow server. The connections must be **JDBC**, and their names should end with the suffix '_JDBC'. The FME Flow server needs at least *db_reader* permissions on the linked databases to read the metadata tables.  

    ```{seealso}
	How to {ref}`RN3_Generic_Update_Reference.md-how-to-create-database-connection`  
	How to {ref}`RN3_Generic_Update_Reference.md-how-to-add-fme-database-user`  
	``` 

- The database connection parameters may contain both non-JDBC and JDBC versions of the connection name. The non-JDBC connection names must match the JDBC one, just without the '_JDBC' suffix. The data harvesting process automatically creates attributes with the JDBC versions of the database connections if the respective parameters contain non-JDBC ones.  
- The RN3 metadata tables DataCollection and ReportingDataset referred to in the **[metadata_Table_DataCollection]** and **[metadata_Table_ReportingDataset]** parameters are currently not used by the data harvesting process. The columns were added to the Harvesting Parameters table during the initial stages of the process development, when it was assumed they would be used. They remain there and are treated as mandatory for historical reasons and in case they are needed in future versions.  

##### Attachment harvesting

Attachments will be harvested only if the **[attachmentParentPath]** value is provided and the data download format is *CSV* or *parquet*.  
```{seealso}
See **etlExportVersion** in FME workspace {ref}`RN3_Generic_Update_Reference.md-fme-workspace-user-parameters` for how to decide and specify the download file format.
```

Standard **[attachmentParentPath]** value is:  
```
'\\cwsfileserver.eea.dmz1\projects\ReportnetResources\RN3_GenericHarvesting_data\RN3_attachments\'
```

We suggest using it, as the FME Flow server already has proper permissions to this CWS folder. If you use a different folder, ensure FME has permission to create subfolders and add files.  

```{warning}
The RN3 data collection with the [attachmentParentPath] field populated must not contain a table named **attachments**. The RN3 export API reserves that name for the folder where it stores the attachment files. Data files from any table named attachments will therefore be handled as attachment files rather than data files, and ultimately, this data will not be extracted and stored in the Harvested Data tables.  
```

The attachment files will be imported as zip files into the following directory structure under the parent folder specified in the [attachmentParentPath]:  
```
\<obligationId>\<dataflowId>\<dataProviderCode>\<dataCollectionId>\<snapshotId>\<rn3TableName>\<fieldName>\<recordId>.zip
```

The name of the file in the zip file will match the value in the respective attachment field (the name of the file the data provider uploaded to RN3).

The data harvesting process will create a metadata table **[AttachmentFilesMapping]** (if it doesn't already exist) in the same database and under the same table schema as the Harvesting Jobs table.  It adds a record for each harvested and stored attachment file. This information can be used in relevant SQL queries and to locate the specific files in the folder structure.
The table contains these columns:  
- **[obligationId]**
- **[dataflowId]** 
- **[dataflowName]**
- **[dataCollectionId]**
- **[attachment_tableName_rn3]** - Name of the RN3 table the attachment file was harvested from.
- **[attachment_tableName_sql]** - Name of the corresponding Harvested Data table.
- **[attachment_fieldName]** - Name of the corresponding attachment field/column.
- **[rn3_dataProviderCode]** - Code of the data provider.
- **[rn3_snapshotId]** - Identifier of the release snapshot.
- **[rn3_recordId]** - The RN3 identifier of the record the harvested file was attached to.
- **[attachment_fileName]** - Name of the attachment file.
- **[attachment_zipFilePath]** - Full path of the zip file with the attachment, except for the attachmentParentPath part.

#### How to

(RN3_Generic_Update_Reference.md-tables-how-to-create-harvesting-parameters-table)=
##### Create [metadata].[HarvestingParameters] table

**SQL - example:**  

~~~~sql
CREATE TABLE [RN3].[metadata].[HarvestingParameters](
	[obligationId] [int] NOT NULL,
	[dataflowId] [bigint] NOT NULL,
	[dataflowName] [nvarchar](255) NULL,
	[dataCollectionId] [bigint] NULL,
	[metadata_DatabaseConnection] [nvarchar](255) NOT NULL,
	[metadata_Database] [nvarchar](255) NOT NULL,
	[metadata_Schema] [nvarchar](255) NOT NULL,
	[metadata_Table_Dataflow] [nvarchar](255) NOT NULL,
	[metadata_Table_DataCollection] [nvarchar](255) NOT NULL,
	[metadata_Table_HistoricRelease] [nvarchar](255) NOT NULL,
	[metadata_Table_ReportingDataset] [nvarchar](255) NOT NULL,
	[dataflowTables_DatabaseConnection] [nvarchar](255) NOT NULL,
	[dataflowTables_Database] [nvarchar](255) NOT NULL,
	[dataflowTables_Schema] [nvarchar](255) NOT NULL,
	[dataflowTables_Table] [nvarchar](255) NOT NULL,
	[harvestingJobs_DatabaseConnection] [nvarchar](255) NOT NULL,
	[harvestingJobs_Database] [nvarchar](255) NOT NULL,
	[harvestingJobs_Schema] [nvarchar](255) NOT NULL,
	[harvestingJobs_Table] [nvarchar](255) NOT NULL,
	[attachmentParentPath] [nvarchar](500) NULL
) ON [PRIMARY]

~~~~

(RN3_Generic_Update_Reference.md-tables-how-to-populate-harvesting-parameters-table)=
##### Populate [metadata].[HarvestingParameters] table

**SQL - example:**  

~~~~sql
INSERT INTO [RN3].[metadata].[HarvestingParameters]
    ( [obligationId]
    ,[dataflowId]
    ,[dataflowName]
    ,[dataCollectionId]

    ,[metadata_DatabaseConnection]
    ,[metadata_Database]
    ,[metadata_Schema]
    ,[metadata_Table_Dataflow]
    ,[metadata_Table_DataCollection]
    ,[metadata_Table_HistoricRelease]
    ,[metadata_Table_ReportingDataset]

    ,[dataflowTables_DatabaseConnection]
    ,[dataflowTables_Database]
    ,[dataflowTables_Schema]
    ,[dataflowTables_Table]

    ,[harvestingJobs_DatabaseConnection]
    ,[harvestingJobs_Database]
    ,[harvestingJobs_Schema]
    ,[harvestingJobs_Table]

    ,[attachmentParentPath] )

VALUES
-- Dataflow name
-- Data collection name
    (
    <obligationId>, <dataflowId>, '<dataflowName>', <dataCollectionId>
    
    , '<RN3_database_FME_connection_name>'
    , '<RN3_database_name>'
    , 'metadata', 'Dataflow', 'DataCollection', 'HistoricRelease', 'ReportingDataset'
    
    , '<RN3_database_FME_connection_name>'
    , '<RN3_database_name>'
    , 'metadata', 'DataflowTables'
    
    , '<Import_database_FME_connection_name>'
    , '<Import_database_name>'
    , 'metadata', 'HarvestingJobs'
    
    , '\\cwsfileserver.eea.dmz1\system\RN3_attachment\'
    ),

-- Dataflow name
-- Data collection name
    (
    <obligationId>, <dataflowId>, '<dataflowName>', <dataCollectionId>
    
    , '<RN3_database_FME_connection_name>'
    , '<RN3_database_name>'
    , 'metadata', 'Dataflow', 'DataCollection', 'HistoricRelease', 'ReportingDataset'
    
    , '<RN3_database_FME_connection_name>'
    , '<RN3_database_name>'
    , 'metadata', 'DataflowTables'
    
    , '<Import_database_FME_connection_name>'
    , '<Import_database_name>'
    , 'metadata', 'HarvestingJobs'
    
    ,NULL
    )

~~~~


<hr class="thick">


(RN3_Generic_Update_Reference.md-fme-workspace)=
## FME workspace 

The metadata harvesting FME workspace does the following: 
- Processes user parameters and selects relevant records from the **Harvesting Parameters** table.  
- Creates JDBC versions of database connections if the parameters provide non-JDBC versions.  
- Selects relevant records from the **Dataflow Tables** and **Geometry Fields** tables.  
- Creates **Harvested Data** table schema and tables if they don't exist yet.  
- Reads the data schema from the **Template** tables.  
- Creates **Harvesting Jobs** table schema if it doesn't exist yet.  
- Checks if the **Harvesting Jobs** table exists. If it doesn't, creates it by copying relevant release snapshot records from the [HistoricalReleases] RN3 metadata table, or adds new release snapshot records if it does exist.  
- Selects release snapshot records from the **Harvesting Jobs** table that have not been harvested.  
- Downloads the latest RN3 Data collections data from the data providers whose release snapshots have been selected.  
- Reads the downloaded data and inserts it into the **Harvested Data** tables using the dynamic data schema.  
- Processes the downloaded **attachment files** and stores them in the folder structure under the specified parent folder, if specified. Creates and updates the [AttachmentFilesMapping] table as needed.
- Converts the **geometry data**, if specified, and uploads it to the **Harvested Data** tables.
- Documents the result of the data harvesting in the **Harvesting Jobs** table.


**Latest version:**  
<https://fme.discomap.eea.europa.eu/fmeserver/workspaces/run/Dataflows_RN3_Generic_Processes/RN3_Generic_Harvesting_Data_v3a.fmw>

(RN3_Generic_Update_Reference.md-fme-workspace-user-parameters)=
### User parameters

#### Reference

**Mandatory parameters:**  
- **obligationIds** - A comma-separated list of reporting obligation identifiers for which the latest RN3 dataflow should be checked for new releases and harvested.  
- **dataflowIds** - A comma-separated list of all RN3 dataflows that are supposed to be checked for new releases, in all their data collections, and harvested.  
- **dataCollectionIds** -  A comma-separated list of selected RN3 data collections that should be harvested.  
- **HP_databaseConnection** - Name of the FME Database connection linked to the MS SQL database containing the Harvesting Parameters table. It must be a **JDBC** MSSQL connection.
 - **HP_database** - Name of the MS SQL database containing the Harvesting Parameters table (standard is 'RN3'). 
 - **HP_schema** - Name of the Harvesting Parameters table schema (standard is 'metadata'). 
 - **HP_table** - Name of the Harvesting Parameters table (standard is 'HarvestingParameters').
- **baseUrl** - The base URL of the specific Reportnet 3 platform's API service (e.g., https://api.reportnet.europa.eu).
- **etlExportVersion** - The 'version' identifier of the etlexport API endpoint used to download the data collection data from the RN3. Allowed values: 'v3', 'v4', 'v5'.

**Optional parameters:**  
- **snapshotId** - The identifier of a single release snapshot that should be harvested.

#### Explanation

- Only one of the **obligationIds**, **dataflowIds**, or **dataCollectionIds** parameters need to be provided. If more than one is provided, the dataflowIds takes precedence over obligationIds, and dataCollectionIds over dataflowIds. 
- If **dataCollectionIds** is the only parameter of the three provided, the process will harvest data only from the latest of the dataflows with the specific obligationId. It will ignore older dataflows with the same obligationId.
- The **dataflowIds** parameter is the one used most. However, it needs to be used instead of the obligationIds only in cases when an obligation has multiple dataflows in RN3, and we want to harvest data from an older dataflow instead of the latest one, or if multiple dataflows of the same obligation should be harvested at the same time.
- The **dataCollectionIds** is to be used if a dataflow contains more data collections, and only some of them should be harvested by this workspace. If it's left empty, the workspace will harvest data from all data collections of all specified dataflows. If the list is provided, the workspace will harvest only data from the listed collections.
- If **snapshotId** is provided, it must be from a dataflow and a collection listed in the **dataflowIds** and **dataCollectionIds** parameters. The snapshot that has already been successfully harvested will be re-harvested,  and the **[harvestDate]** and **[jobSummary]** of the corresponding Harvesting Job record will be updated. The main purpose of this parameter is to test and analyse why a specific snapshot fails harvesting and import.  
- In the **HP_** paramaters we specify the name and location of the **Harvesting Parameters table**. This is where we can identify whether the location, schema or name is non-standard.
- If the database connection referred to in the **HP_databaseConnection** parameter is not a JDBC connection, the harvesting process will fail.  
- The FME connection referred to in the **HP_databaseConnection** parameter must exist in the EEA's FME Flow server. The connection must be non-JDBC. The FME Flow server needs *db_ddladmin* or *db_owner* permissions on the linked database to access it and write to the metadata tables.  

    ```{seealso}
	How to {ref}`RN3_Generic_Update_Reference.md-how-to-create-database-connection`  
	How to {ref}`RN3_Generic_Update_Reference.md-how-to-add-fme-database-user`  
	``` 

- The **etlExportVersion** specifies the format of the file downloaded by the RN3 API export endpoint:  
    - **v3** - JSON. Geometries are downloaded as extended GeoJSON strings. This version can not be used to harvest attachment files.  
    - **v4** - CSV. Geometries are downloaded as extended GeoJSON strings. Attachment files are included in the download if the [attachmentParentPath] parameter is specified in the corresponding Harvesting Parameters table record.  
    - **v5** - parquet. Geometries are downloaded as EWKB values. Attachment files are included in the download if the [attachmentParentPath] parameter is specified in the corresponding Harvesting Parameters table record.  

<hr>

(RN3_Generic_Update_Reference.md-fme-workspace-schedule)=
### Schedule

The data harvesting processes are commonly run on a regular schedule. This can be achieved by creating an FME Schedule (or Automation) that triggers the FME workspace at specified intervals and supplies it with corresponding user parameters.  
The most commonly used frequency for RN3 metadata harvesting is once a day. It's highly recommended to run it late at night.  
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
