# RN3 Generic Harvesting - Data

___

**Complexity:**  
<span class="stars">★★★★⯪</span>


**Description:**  
Data harvesting is an FME-orchestrated process that downloads **data** of selected **Reportnet 3 dataflows** and stores it in a **series of tables** in an **MS SQL database**.

The process can be triggered manually or, more commonly, scheduled to run regularly.

The **prerequisite** for the process is existance of metadata tables regularly updated by a **RN3 Generic Harvesting - Metadata** process.  

The process setup is complex and includes creation and population of multiple additional SQL tables.

## Quick setup

<ins>If ...:</ins>  
1. Text.  
	- See {ref}`reference` for text.   


<ins>If ...:</ins>  
1. Text.  
	- See {ref}`reference` for text.  
2. Text.   
	- See text {ref}`reference` for text.  
	- See text {ref}`reference` for text.  


(RN3_Generic_Harvesting_Data.md-database)=
## Databases

### RN3 Database

text


### Import Database

text


(RN3_Generic_Harvesting_Data.md-tables)=
## Tables

text

(RN3_Generic_Harvesting_Data.md-tables-harvesting-parameters)=
### HarvestingParameters

text

(RN3_Generic_Harvesting_Data.md-tables-harvesting-parameters-reference)=
#### Reference

text

(RN3_Generic_Harvesting_Data.md-tables-harvesting-parameters-explanation)=
#### Explanation

text

#### How to

(RN3_Generic_Harvesting_Data.md-tables-tables-how-to-create-harvesting-parameters-table)=
##### Create [metadata].[HarvestingParamaters] table

**SQL - example:**  


~~~~sql
Select * from table

~~~~



(RN3_Generic_Harvesting_Data.md-tables-dataflow-tables)=
### DataflowTables

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

(RN3_Generic_Harvesting_Data.md-tables-dataflow-tables-geomfields)=
### DataflowTables_GeometryFields

text

(RN3_Generic_Harvesting_Data.md-tables-dataflow-tables-geomfields-reference)=
#### Reference

text

(RN3_Generic_Harvesting_Data.md-tables-dataflow-tables-geomfields-explanation)=
#### Explanation

text

#### How to

(RN3_Generic_Harvesting_Data.md-tables-tables-how-to-create-dataflow-tables-geomfields-table)=
##### Create [metadata].[DataflowTables_GeometryFields] table

**SQL - example:**  


~~~~sql
Select * from table

~~~~

(RN3_Generic_Harvesting_Data.md-tables-templates)=
### Template tables

text

(RN3_Generic_Harvesting_Data.md-tables-templates-reference)=
#### Reference

text

(RN3_Generic_Harvesting_Data.md-tables-templates-explanation)=
#### Explanation

text

#### How to

text


(RN3_Generic_Harvesting_Data.md-tables-harvesting-jobs)=
### HarvestingJobs

text

(RN3_Generic_Harvesting_Data.md-tables-harvesting-jobs-reference)=
#### Reference

text

(RN3_Generic_Harvesting_Data.md-tables-harvesting-jobs-explanation)=
#### Explanation

text

#### How to

text


(RN3_Generic_Harvesting_Data.md-tables-harvesting-jobs)=
### Harvested data tables

text

(RN3_Generic_Harvesting_Data.md-tables-harvesting-jobs-reference)=
#### Reference

text

(RN3_Generic_Harvesting_Data.md-tables-harvesting-jobs-explanation)=
#### Explanation

text


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
