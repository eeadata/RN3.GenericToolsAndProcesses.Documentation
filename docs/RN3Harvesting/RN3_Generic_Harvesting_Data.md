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
The process can harvest only data from the latest release. If a data provider made multiple releases between two harvesting jobs, data from their earlier releases won't be harvested.  
Older releases also can't be reharvested. The data will be replaced with data from the latest release instead.  

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

The data harvesting setup is flexible with respect to the names and number of databases. The standard setup model, however, works with two databases.  
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

This documentation uses standard names for tables and schemas involved in the data harvesting process. The responsible data manager can, however, decide to use different names if needed.


(RN3_Generic_Harvesting_Data.md-tables-templates)=
### Template tables

Creating template tables should be the first step in the setup.  
Each RN3 dataflow table that should have its data harvested must have a template table in the Import database.  

The template tables must be left empty.  

The harvesting process uses the template tables as the source of dynamic schemas when writing the data to the database.  
When it runs for the specific dataflow for the first time, it also uses them to create the harvested data tables (see {ref}`RN3_Generic_Harvesting_Data.md-tables-harvested-data`).
 
#### Schema and names

The standard name for the template table schema is **[template]**.

The names of the template tables should match the names of the RN3 tables. 
If the dataflow, however, contains multiple tables with the same names, just in different reporting dataset schemas, the template tables need to be differentiated. The standard approach is to prefix the template table name with an abbreviation of the specific RN3 dataset schema.

```{admonition} Example
:class: dropdown
The Habitatas directive reporting dataflow contains tables with the same names in its reporting dataset schemas. 
For example, the table *Maps* is present in both the *Reporting data - Habitats* and *Reporting data - Species* datasets. In the NatureArt17_Import database, the corresponding template tables are named *Habitats_Maps* and *Species_Maps*. 
For the sake of consistency, all template table names were prefixed with an abbreviation of their dataset schema.
```

Another, relatively simple option for handling tables with the same names is to use different table schemas for tables from different datasets.  
(There may be other options, like different dataset-specific import databases or data-collection-specific harvesting processes, but these may be unnecessarily complex, and we are not going to describe them.)

#### Structure

The structure of the template table should match that of the RN3 dataset table. Table columns must be named the same as the RN3 table fields (including the same letter case). 

In addition to the data columns, each template table must contain these 3 **metadata columns**:
- **[rn3_dataProviderCode]**
- **[rn3_snapshotId]**
- **[rn3_recordId]**
```{seealso}
See {ref}`RN3_Generic_Harvesting_Data.md-tables-templates-reference` for the details
```
The harvesting process will fail if any metadata column is missing.

The harvesting process will not fail if the template tables contain misnamed data columns or are missing any data columns. The misnamed columns will not be populated, though, and data from the missing columns will not be imported.

Each data column needs a data type appropriate for the data in the corresponding RN3 fields.

```{warning}
The harvesting process will fail if the template table column has an inappropriate data type. This includes text columns (nvarachar) with an insufficient character limit.  
To continue harvesting from the RN3 dataflow, the data type must be corrected in both the affected template and the corresponding harvested data table, or harvesting of the specific release must be disabled.
```

#### Geometry data

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

text

(RN3_Generic_Harvesting_Data.md-tables-harvesting-jobs)=
### Harvesting jobs tables

text

(RN3_Generic_Harvesting_Data.md-tables-harvesting-jobs-reference)=
#### Reference

text

(RN3_Generic_Harvesting_Data.md-tables-harvesting-jobs-explanation)=
#### Explanation

text

#### How to

text

(RN3_Generic_Harvesting_Data.md-tables-dataflow-tables)=
### RN3 Dataflow metadata tables


(RN3_Generic_Harvesting_Data.md-tables-dataflow-tables)=
### Dataflow tables table

text

(RN3_Generic_Harvesting_Data.md-tables-dataflow-tables-reference)=
#### Reference

text

(RN3_Generic_Harvesting_Data.md-tables-dataflow-tables-explanation)=
#### Explanation

text

#### How to

(RN3_Generic_Harvesting_Data.md-tables-tables-how-to-create-dataflow-tables-table)=
##### Create [metadata].[DataflowTables] table

**SQL - example:**  


~~~~sql
Select * from table

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
- If you don't have permissions to create a Schedule, or are not confident enough to create it, ask EEA Service Desk, or a colleague with appropriate permissions and experience, to create it for you.
