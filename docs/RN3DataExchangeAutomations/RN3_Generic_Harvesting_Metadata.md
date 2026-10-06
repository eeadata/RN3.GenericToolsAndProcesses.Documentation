(rn3-generic-harvesting-metadata)=
# RN3 Generic Harvesting - Metadata

<hr class="double">

**Complexity:**  
<span class="stars">★★⯪☆☆</span>


## Introduction

Metadata harvesting is an FME-orchestrated process that downloads **metadata** of selected **Reportnet 3 dataflows** and stores it in a **series of tables** in an **MS SQL database**.

The process can be triggered manually or, more commonly, scheduled to run regularly.

The harvested dataflow metadata can be used for multiple purposes, but the main one is for use in RN3 data harvesting. 
It is a **prerequisite** for the **{ref}`rn3-generic-harvesting-data`** process.  

<hr class="thick">

## Quick setup

<ins>If the RN3 database already exists on your MS SQL server and has been used for metadata harvesting:</ins>  
1. For each RN3 dataflow you want to harvest, **insert a record** with the required values into the **[RN3].[metadata].[Dataflow] table**.  
	- See How to {ref}`RN3_Generic_Harvesting_Metadata.md-tables-how-to-populate-metadata-dataflow-table`.   
	- See Tables {ref}`RN3_Generic_Harvesting_Metadata.md-tables-reference` and {ref}`RN3_Generic_Harvesting_Metadata.md-tables-explanation` for more details.  

<ins>If the RN3 database doesn't exist on your MS SQL server:</ins>  
1. Ask EEA's Service Desk to **create the RN3 database**.  
	- See {ref}`RN3_Generic_Harvesting_Metadata.md-database` for more details and alternatives.  
2. **Create [metadata].[Dataflow] table** in the RN3 database.   
	- See How to {ref}`RN3_Generic_Harvesting_Metadata.md-tables-how-to-create-metadata-dataflow-table`.  
	- See {ref}`RN3_Generic_Harvesting_Metadata.md-tables` for more details and alternatives.  
3. For each RN3 dataflow you want to harvest, **insert a record** with the required values into the **[RN3].[metadata].[Dataflow] table**.  
	- See How to {ref}`RN3_Generic_Harvesting_Metadata.md-tables-how-to-populate-metadata-dataflow-table`.  
	- See Tables {ref}`RN3_Generic_Harvesting_Metadata.md-tables-reference` and {ref}`RN3_Generic_Harvesting_Metadata.md-tables-explanation` for more details.  
4. **Create a metadata harvesting schedule** on the EEA's FME Flow server.  
	- See {ref}`RN3_Generic_Harvesting_Metadata.md-fme-workspace-schedule` for the details.  
	- See {ref}`RN3_Generic_Harvesting_Metadata.md-fme-workspace` for the details on the FME workspace.  

<hr class="thick">

(RN3_Generic_Harvesting_Metadata.md-database)=
## Database

The MS SQL database where the harvesting process stores the metadata of RN3 dataflows is traditionally a dedicated database named **RN3**.   
It can, however, be any database.

The responsible data manager must specify what database the harvesting process should use in the {ref}`RN3_Generic_Harvesting_Metadata.md-fme-workspace-user-parameters` of the FME workspace.

*In this documentation, we will continue referring to this database as the **RN3** database.*

```{important}
Please contact EEA Service Desk if you want to create a new MS SQL database.
```
```{important}
If harvested metadata are to be used in the generic data harvesting process, the RN3 database must be on the same MS SQL Server as the database for the harvested data (traditionally referred to as the **Import** database).  
```
If more data managers (data custodians) use the same MS SQL Server to manage different dataflows, they can all use the same RN3 database.  
In such situations, data managers must take care not to affect the metadata of other dataflows when setting up their metadata harvesting. They should also appoint one of them to manage the metadata harvesting schedule, since the process requires only a single schedule to run for all dataflows with metadata in the RN3 database.

<hr class="thick">

(RN3_Generic_Harvesting_Metadata.md-tables)=
## Tables

By default, the harvesting process stores RN3 dataflow metadata in the RN3 database in the following tables under the **[metadata]** table schema:  
- **[Dataflow]**  
- **[DataCollection]**  
- **[DesignDataset]**  
- **[EUDataset]**  
- **[HistoricRelease]**  
- **[HistoricRelease_statusLog]**  
- **[ReferenceDataset]**  
- **[ReportingDataset]**  

If needed, the default table schema and table names can be changed by modifying the respective {ref}`RN3_Generic_Harvesting_Metadata.md-fme-workspace-user-parameters` of the FME workspace.  

*In this documentation, we will continue using the default table schema and table names.*  

The table schema and all the tables are created by the harvesting process when it runs in the RN3 database for the first time.  
No metadata tables will be populated during this first execution because the process doesn't yet know what to harvest. The responsible data manager must provide this information in the **[metadata].[Dataflow]** table.  

The data manager can skip the initial execution of the harvesting process and create the **[metadata]** schema and the **[metadata].[Dataflow]** table themselves.  

```{seealso}
See How to {ref}`RN3_Generic_Harvesting_Metadata.md-tables-how-to-create-metadata-dataflow-table`  
```

Once the [metadata].[Dataflow] table has been created, the responsible data manager must insert a new record for each RN3 dataflow they want to harvest metadata from. Only four values need to be provided in each record. The harvesting process will fill the rest.  

```{seealso}
See How to {ref}`RN3_Generic_Harvesting_Metadata.md-tables-how-to-populate-metadata-dataflow-table`  
```

The content of all tables is updated every time the harvesting process is executed. The new records are added, and existing records are overwritten. The exception is [HistoricRelease_statusLog] table (see Tables {ref}`RN3_Generic_Harvesting_Metadata.md-tables-explanation` for clarification).  

(RN3_Generic_Harvesting_Metadata.md-tables-reference)=
### Reference

#### [Dataflow]

Besides the harvested metadata of the RN3 dataflow itself, the table contains a few columns that need to be prefilled before harvesting can start.  
Description of selected columns:  
- **[dataflowId]** - The RN3 dataflow identifier. The primary key. Must be prefilled.  
- **[obligationId]** - Reporting obligation identifier. Must be prefilled.  
- **[harvestMetadata]** - A boolean value indicating whether the dataflow metadata should actually be harvested (1=yes, 0=no). Must be prefilled.  
- **[apiKey]** - The API-key of the dataflow. Must be prefilled.  

#### [DesignDataset]

Contains harvested metadata of all dataflow's design datasets.  
Description of selected columns:  
- **[datasetId]** - The RN3 dataset identifier. The primary key.  
- **[dataflowId]** - The foreign key linking the [DesignDataset] record to the corresponding [Dataflow] record.  
- **[datasetSchema]** - The unique identifier of the dataset schema.   It's the same for all datasets created from the same schema.  
- **[datasetSchemaJson]** - A JSON string containing all design dataset metadata which is not extracted by the harvesting process. This includes a list of all dataset fields and their metadata.  

#### [ReferenceDataset]

Contains harvested metadata of all dataflow's reference datasets.  
Description of selected columns:  
- **[datasetId]** - The RN3 dataset identifier. The primary key.  
- **[dataflowId]** - The foreign key linking the [DesignDataset] record to the corresponding [Dataflow] record.  
- **[datasetSchema]** - The unique identifier of the dataset schema.   It's the same for all datasets created from the same schema.  

#### [ReportingDataset]

Contains harvested metadata of all dataflow's reporting datasets (the data provider's datasets).  
Description of selected columns:  
- **[datasetId]** - The RN3 dataset identifier. The primary key.   
- **[dataflowId]** - The foreign key linking the [ReportingDataset] record to the corresponding [Dataflow] record.  
- **[datasetSchema]** - The unique identifier of the dataset schema.   It's the same for all datasets created from the same schema.  
- **[isReleased]** - A boolean field indicating whether the data provider has released their data.  
- **[dataProviderId]** - A numeric identifier of the data provider.  
- **[status]** - Status of the latest data provider release.  

 #### [DataCollection]

Contains harvested metadata of all dataflow's data collections.  
Description of selected columns:  
- **[dataCollectionId]** - The RN3 dataset identifier. The primary key.  
- **[dataflowId]** - The foreign key linking the [DataCollection] record to the corresponding [Dataflow] record.  
- **[datasetSchema]** - The unique identifier of the dataset schema.   It's the same for all datasets created from the same schema.  

#### [EUDataset]

Contains harvested metadata of all dataflow's EU datasets.  
Description of selected columns:  
- **[datasetId]** - The RN3 dataset identifier. The primary key.  
- **[dataflowId]** - The foreign key linking the [EUDataset] record to the corresponding [Dataflow] record.  
- **[datasetSchema]** - The unique identifier of the dataset schema.  

#### [HistoricRelease]

Contains harvested metadata of dataset release snapshots.  
Description of selected columns:  
- **[snapshotId]** - The RN3 release snapshot identifier. The primary key.  
- **[datasetId]** - The foreign key linking the [HistoricRelease] record to the corresponding [ReportingDataset] record.  
- **[dataCollectionId]** - The foreign key linking the [HistoricRelease] record to the corresponding [DataCollection] record.  
- **[dataflowId]** - The foreign key linking the [HistoricRelease] record to the corresponding [Dataflow] record.  
- **[dataProviderCode]** - A code of the data provider (e.g., a two-letter country code).  
- **[dateReleased]** - A timestamp of the corresponding dataset release.  
- **[dcrelease]** - A boolean value indicating if this release snapshot is the latest release snapshot from the reporting dataset of the given data provider (1) or if it's one from the older releases (0). The latest snapshot is the one currently used in the corresponding Data collection.  
- **[eurelease]** - A boolean value indicating if this release snapshot is the one currently used in the corresponding EU dataset.  
- **[restrictFromPublic]** - A boolean value indicating if this release snapshot was restricted by the data provider from public view at its release.  

#### [HistoricRelease_statusLog]

Combines time-specific information of release snapshot with the information on release status from the corresponding Reporting dataset.  
Description of selected columns:  
- **[snapshotId]** - The foreign key linking the record to the corresponding [HistoricRelease] record.  
- **[status]** - The latest status of the data provider release the release snapshot is part of.  

(RN3_Generic_Harvesting_Metadata.md-tables-explanation)=
### Explanation

- The value in the **apiKey** column of the **[Dataflow]** table is constructed by prefixing the dataflow API-key with the 'ApiKey ' string (including the space!).  
  
	The dataflow's **API key** can be obtained from the dataflow page by following the instructions at <https://eea.github.io/eea.help.reportnet3/rest-api/>.  

- The **[HistoricRelease_statusLog]** table has been designed as a workaround to solve a problem caused by a faulty architecture of the RN3 metadata database. In this architecture, the release status is not an attribute of the release itself, but of the reporting dataset.  

	The release status, which is very important for the further use of the data, is the status assigned to the dataset by the data requester at the end of the **Final Feedback** process. This status indicates if the release has been technically accepted or a correction has been requested.  
	
	But because a data provider can release their data many times, the final feedback status may be replaced by a new status when a new release is made. This made it difficult to identify release snapshots that were technically accepted.  
	
	The [HistoricRelease_statusLog] table has been designed to preserve the latest release status for each release snapshot. The metadata harvesting process updates the record as long as the corresponding [HistoricRelease] record has [dcrelease] = 1 (meaning it's the latest release snapshot). When the data provider makes a new release, the [dcrelease] value of the [HistoricRelease] record changes to 0, and the corresponding [HistoricRelease_statusLog] record is no longer updated by the metadata harvesting process.  


### How to

(RN3_Generic_Harvesting_Metadata.md-tables-how-to-create-metadata-dataflow-table)=
#### Create [metadata].[Dataflow] table

**SQL - example:**  

```{warning}
Run this only if the table doesn't exist yet, or you are the only data manager using the RN3 database.
```
~~~~sql
USE [RN3]

-- DROP TABLE [metadata].[Dataflow]

CREATE TABLE [metadata].[Dataflow](
	[dataflowId] [bigint] NOT NULL,
	[name] [nvarchar](4000) NULL,
	[description] [nvarchar](4000) NULL,
	[deadlineDate] [datetime] NULL,
	[releasable] [bit] NULL,
	[obligationId] [int] NOT NULL,
	[harvestMetadata] [bit] NOT NULL,
	[apiKey] [nvarchar](100) NOT NULL,
	[json] [nvarchar](max) NULL,
	[recordLastModified] [datetime] NULL,
	[obligationScope] [nvarchar](100) NULL,
	[obligationAlias] [nvarchar](100) NULL,
	[obligationDisclaimerText] [nvarchar](max) NULL,
	[manualAcceptance] [bit] NULL,
 CONSTRAINT [PK_Dataflow] PRIMARY KEY CLUSTERED 
(
	[dataflowId] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON, OPTIMIZE_FOR_SEQUENTIAL_KEY = OFF) ON [PRIMARY]
) ON [PRIMARY] TEXTIMAGE_ON [PRIMARY]
~~~~

(RN3_Generic_Harvesting_Metadata.md-tables-how-to-populate-metadata-dataflow-table)=
#### Populate [metadata].[Dataflow] table

**SQL - example:**  

~~~~sql
USE [RN3]

-- DELETE FROM [metadata].[Dataflow] WHERE [dataflowId] = <dataflowId>

INSERT INTO [metadata].[Dataflow] (
	[dataflowId]
	,[obligationId]
	,[harvestMetadata]
	,[apikey]
	) 
VALUES
	(<dataflowId>,<obligationId>,1,'ApiKey <dataflow API Key>'),
	(<dataflowId>,<obligationId>,1,'ApiKey <dataflow API Key>'),
	(...)
~~~~

<hr class="thick">

(RN3_Generic_Harvesting_Metadata.md-fme-workspace)=
## FME workspace 

The metadata harvesting FME workspace does the following: 
- Creates [metadata] table schema if it doesn't exist.
- Creates the [metadata] tables if they don't exist.
- Selects [metadata].[Dataflow] records where [harvestMetadata] = 1.
- Uses the selected values to get the dataflow metadata from Reportnet 3 as JSON strings
- Parses the JSON, extracts relevant metadata values and writes them in the corresponding tables using the Upsert action.

**Latest version:**  
<https://fme.discomap.eea.europa.eu/fmeserver/workspaces/run/Dataflows_RN3_Generic_Processes/RN3_Generic_Harvesting_Metadata_v2.fmw>

(RN3_Generic_Harvesting_Metadata.md-fme-workspace-user-parameters)=
### User parameters

#### Reference

**Mandatory parameters:**  
- **ApiUrl** - The base URL of the specific Reportnet 3 platform's API service (e.g., https://api.reportnet.europa.eu).  
- **RN3_metadata_databaseConnection** - Name of the FME Database connection linked to **RN3 database**.   
- **RN3_metadata_database** - Name of the RN3 database (default value is 'RN3').  
- **RN3_metadata_schema** - Name of the table schema for the metadata tables (default value is 'metadata').  
- **RN3_metadata_table_Dataflow** - Name of the table for dataflow metadata (default value is 'Dataflow').  
- **RN3_metadata_table_DataCollection** - Name of the table for data collection metadata (default value is 'DataCollection').  
- **RN3_metadata_table_HistoricRelease** - Name of the table for release snapshot metadata (default value is 'HistoricRelease').  
- **RN3_metadata_table_ReportingDataset** - Name of the table for reporting dataset metadata (default value is 'ReportingDataset').  

**Optional parameters:**  
- **RN3_metadata_table_DesignDataset** -  Name of the table for design dataset metadata (default value is 'DesignDataset').  
- **RN3_metadata_table_ReferenceDataset** - Name of the table for reference dataset metadata (default value is 'ReferenceDataset').  
- **RN3_metadata_table_EUDataset** - Name of the table for EU dataset metadata (default value is 'EUDataset').  

#### Explanation
- The **optional parameters** represent tables that were added to the metadata harvesting in later versions of the FME workspace. Making them optional was intended to ensure that data managers do not have to change user parameter values in existing schedules and automations. If optional parameters are not provided, the harvesting process uses default names for the respective metadata tables.   
- The FME connection referred to in the **RN3_metadata_databaseConnection** parameter must exist in the EEA's FME Flow server. The connection must be non-JDBC. The FME Flow server needs *db_ddladmin* or *db_owner* permissions on the linked database to access it and write to the metadata tables.  

    ```{seealso}
	How to {ref}`RN3_Generic_Harvesting_Metadata.md-how-to-create-database-connection`  
	How to {ref}`RN3_Generic_Harvesting_Metadata.md-how-to-add-fme-database-user`  
	``` 

(RN3_Generic_Harvesting_Metadata.md-fme-workspace-schedule)=
### Schedule

The metadata harvesting processes are commonly run on a regular schedule. This can be achieved by creating an FME Schedule (or Automation) that triggers the FME workspace at specified intervals and supplies it with corresponding user parameters.  
The most commonly used frequency for RN3 metadata harvesting is two times a day. It's highly recommended that one of the times runs late at night.  
If a scheduled generic data harvesting process depends on the harvested metadata, the metadata harvesting should run about 10-15 minutes before to allow it to finish.

```{seealso}
How to {ref}`RN3_Generic_Harvesting_Metadata.md-how-to-create-fme-schedule`  
``` 

### How to

(RN3_Generic_Harvesting_Metadata.md-how-to-create-database-connection)=
#### Create a database connection on FME Flow

- Follow <https://support.safe.com/hc/en-us/articles/25407463461517-FAQ-Database-Connections-on-FME-Flow>.
- If you don't have permissions to add a connection, or are not confident enough to create it, ask EEA Service Desk to create it for you.

(RN3_Generic_Harvesting_Metadata.md-how-to-add-fme-database-user)=
#### Add FME Flow server as a user to an MS SQL database

- Follow <https://learn.microsoft.com/en-us/sql/relational-databases/security/authentication-access/create-a-database-user?view=sql-server-ver17#create-a-user-with-ssms>.
- Add a user with eeadmz1\fmeservice as the user name and login name.
- Select appropriate permission options on the Membership page.
- If you don't have permissions to add users to the database, ask EEA Service Desk to add it.

(RN3_Generic_Harvesting_Metadata.md-how-to-create-fme-schedule)=
#### Create a Schedule on FME Flow.
- Follow <https://docs.safe.com/fme/html/FME-Flow/WebUI/schedules.htm>.
- If you don't have permissions to create a Schedule, or are not confident enough to create it, ask the EEA Service Desk or a colleague with appropriate permissions and experience to create it for you.
