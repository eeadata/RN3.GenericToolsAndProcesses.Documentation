# RN3 Generic Update - prefill table

___

**Description:**  
This external integration adds an **Import dataset data** option to the RN3 dataset, allowing the data provider to prefill a specific dataset table (***target table***) with selected data (usualy the provider specific data) from a specifc table or view in an MS SQL database (***prefill data source***).  

```{warning}
Any existing records in the *target table* will be automatically deleted before the import.  
```

```{warning}
The process doesn't check if data in the selected *prefill data source* matches the constraints of the *target table* fields (e.g., data type) before the import. The discrepancies will be identified during the RN3 Validation.
```


**Operation type:**  
IMPORT FROM OTHER SYSTEM  


## External integration setup - example

**Name:**  
Prefill table - *BiologyEQRClassificationProcedure*  

**Description:**  
Prefill table - *BiologyEQRClassificationProcedure* (generic)  

**Repository:**  
Dataflows_RN3_Generic_Processes  

**Workspace name:**  
RN3_Generic_Update_prefillTable.fmw  

**Operation:**  
IMPORT FROM OTHER SYSTEM  

**Custom parameters**  

| **Parameter key** | **Parameter value** |
| --- | --- |
| **rn3_tableName** | *BiologyEQRClassificationProcedure* |
| **PS_databaseConnection** | *WISE_SOE_WIGEON* |
| **PS_databaseName** | *WISE_SOE* |
| **PS_tableSchema** | *data* |
| **PS_tableName** | *T_WISE2_BiologyEQRClassificationProcedure* |
| **PS_field_provider** | *countryCode* |
| **PS_field_filter** |
| **filterValue** | 


## External integration custom parameters

### Reference

**Mandatory parameters:**  
- **rn3_tableName** - Name of the RN3 reporting dataset table that should be prefilled with data from the specified *prefill data source*. The ***target table***.  
- **PS_databaseConnection** - Name of the FME Database connection linked to the MS SQL database containing the *prefill data source*.  
- **PS_databaseName** - Name of the MS SQL database containing the *prefill data source*.  
- **PS_tableSchema** - Name of the table schema of the *prefill data source*.  
- **PS_tableName** - Name of the table or view that contains the prefill data. The ***prefill data source***.  

**Optional parameters:**  
- **PS_field_provider** - Name of a field in the *prefill data source*, which contains the data provider code values.  
- **PS_field_filter** - Name of a field in the *prefill data source*, which contains values by which the source data should be filtered. It's used together with the **filterValue** parameter.  
- **filterValue** - see **PS_field_filter**.  

### Explanation

- The ***prefill data source*** should be created in cooperation with the **responsible dataflow owner** (e.g., data steward).  
- The FME connection referred to in the **PS_databaseConnection** parameter must exist in the EEA's FME Flow server. The connection can be either non-JDBC or JDBC. If it's JDBC, the connection name must end with '_JDBC'. The FME Flow server needs at least *db_datareader* permissions in the linked database to access and read the *prefill data source*. 

    ```{seealso}
	How to {ref}`RN3_Generic_Update_prefill_table.md-how-to-create-database-connection`  
	How to {ref}`RN3_Generic_Update_prefill_table.md-how-to-add-fme-database-user`  
	``` 

- The process will import values only from those columns in the *prefill data source* that have the **same name** as the fields in the *target table*. Any additional columns in the *prefill data source* will be ignored. The *target table* fields that do not have a corresponding column in the *prefill data source* will be imported empty. 
	
	```{important}
	If the *target table* contains any **geometry fields**, and the prefill action should prefill those too, the *prefill data source* needs to contain the geometry values formatted as an **Extended GeoJSON** string, not as SQL geometry!  
	This may change in future versions.  
	```

- If the **PS_field_provider** is provided, it's used to select only those *prefill data source* records where the value in the specified field is the same as the RN3 dataset's data provider code value (RN3 supplies the data provider code value to the FME workspace automatically, as the countryCode user parameter). 
	
	```{warning}
	If the *prefill data source* doesn't contain a column with the name matching the **PS_field_provider** value, the import process will fail!  
	```
	
- If both the **PS_field_filter** and the **filterValue** is provided, the process uses only the *prefill data source* records where the value in the specified field is the same as in the **filterValue** parameter.  
	
	```{warning}
	If the *prefill data source* doesn't contain a column with the name matching the **PS_field_filter** value, the import process will fail!  
	```

- If no optional parameters are provided, the process imports all records from the *prefill data source* to the *target table*.  
- If all optional parameters are provided, the process imports to the *target table* only those prefill source records that match both the data provider and the filter value.  

### How to

(RN3_Generic_Update_prefill_table.md-how-to-create-database-connection)=
#### Create a database connection on FME Flow

- Follow <https://support.safe.com/hc/en-us/articles/25407463461517-FAQ-Database-Connections-on-FME-Flow>
- If you don't have permissions to add a connection, or are not confident enough to create it, ask EEA Service Desk to create it for you

(RN3_Generic_Update_prefill_table.md-how-to-add-fme-database-user)=
#### Add FME Flow server as a user to an MS SQL database

- Follow <https://learn.microsoft.com/en-us/sql/relational-databases/security/authentication-access/create-a-database-user?view=sql-server-ver17#create-a-user-with-ssms>
- Add a user with eeadmz1\fmeservice as the user name and login name
- Select appropriate permission options in the Membership page
- If you don't have permissions to add users to the database, ask EEA Service desk to add it


## FME workspace

The external integration FME workspace does the following: 
- Reads data from a defined *prefill data source*.  
- Applies specified filters if provided.  
- Imports the result into the *target table*.  

It transforms and saves the selected *prefill data source* data into a single CSV file, which it then imports to the *target table* using the '/dataset/v2/importFileData/{datasetId}' RN3 API endpoint.

**Latest version:**  
<https://fme.discomap.eea.europa.eu/fmeserver/workspaces/run/Dataflows_RN3_Generic_Processes/RN3_Generic_Update_prefillTable.fmw>