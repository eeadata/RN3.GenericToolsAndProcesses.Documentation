# RN3 Generic Update prefill table
---

**Description:**  
External integration reads data from a defined prefill data source - table/view in a MS SQL database - applies specified filters if provided (usually the data provider code), and imports the result into a selected RN3 reporting dataset table.  

**Operation type:**  
IMPORT FROM OTHER SYSTEM  

**Latest version:**  
[https://fme.discomap.eea.europa.eu/fmeserver/workspaces/run/Dataflows_RN3_Generic_Processes/RN3_Generic_Update_prefillTable.fmw](https://fme.discomap.eea.europa.eu/fmeserver/workspaces/run/Dataflows_RN3_Generic_Processes/RN3_Generic_Update_prefillTable.fmw)  


## External integration setup (example)

**Name:**  
Prefill table - *BiologyEQRClassificationProcedure*  

**Description:**  
Prefill table - *BiologyEQRClassificationProcedure*(generic)  

**Repository:**  
Dataflows_RN3_Generic_Precesses  

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

**Mandatory:**  
- **rn3_tableName** - Name of the RN3 reporting dataset table that should be prefilled with data from the specified prefill source.  
- **PS_databaseConnection** - Name of the FME Database connection linked to the MS SQL database containing the prefill source.  
- **PS_databaseName** - Name of the MS SQL database containing the prefill source.  
- **PS_tableSchema** - Name of the table schema of the prefill source.  
- **PS_tableName** - Name of the table/view that contains the prefill data (prefill data source).  

**Optional:**  
- **PS_field_provider** - Name of a field in the prefill data source, which contains the data provider code values.  
- **PS_field_filter** - name of a field in the prefill data source, which contains values by which the source data should be filtered. It's used together with the filterValue parameter.  
- **filterValue** - see **PS_field_filter**.  


### Explanation

- If **PS_field_provider** is provided, it's used to select only the prefill data source records where the value in the specified field is the same as the RN3 dataset's data provider code value (RN3 supplies the data provider code value to the FME workspace automaticaly, as the countryCode user parameter).
- If both **PS_field_filter** and **filterValue** is provided, the process uses only the prefill data source records where the value in the specified field is the same as the filterValue parameter.
- If no optional parameters are provided, the process imports all records from the prefill data source.
- If all optional parameters are provided, the process imports only those prefill source records that match both the data provider and the filter value.
