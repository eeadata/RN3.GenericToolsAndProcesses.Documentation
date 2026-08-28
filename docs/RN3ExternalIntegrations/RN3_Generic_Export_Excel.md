# RN3 Generic Export - Excel

___

**Complexity:**  
<span class="stars">⯪☆☆☆☆</span>


**Description:**  
This external integration adds an **Export dataset data** option to the RN3 dataset.  
It allows the data provider to Export dataset data into an MS Excel file.  

By default, the process exports data from all RN3 dataset tables to an Excel file, creating one worksheet per table.  
The worksheet names will match the names of the corresponding RN3 tables.  
If an RN3 table contains no data, an empty worksheet will be created.  

```{warning}
If an RN3 table name is longer than the maximum allowed length of an Excel worksheet name, the default process will try to truncate the worksheet name to 31 characters.  
If multiple tables in the RN3 dataset have names starting with the same 31 characters, the resulting worksheets may include unexpected names.  
```

```{note}
To avoid the default behaviour and ensure that all worksheets in the exported Excel file have the desired names, add the **'renameTables'** custom parameter to the external integration setup, where you provide the mapping key between the RN3 table names and the worksheet names where they are different.  

*(The renameTables parameter is available from v3 of the FME workspace.)*  
```

```{warning}
If any of the RN3 tables contain geometry fields and the reported geometries are very large, the export process may fail to read the data or will run slowly. If it succeeds, it removes the geometry data to avoid potential failure when creating the Excel file.  

*(This behaviour is introduced in v3 of the FME workspace.)*  
```

```{note}
If any tables should be excluded from the export, add their names to the **'excludeTables'** custom parameter in the external integration setup. This is recommended for tables with geometry fields and read-only tables.  

*(The excludeTables parameter is available from v3 of the FME workspace.)*  
```

**Operation type:**  
IMPORT  


## External integration setup - example

**Name:**  
Export Excel file  

**Description:**  
Export Excel file (generic)  

**Repository:**  
Dataflows_RN3_Generic_Processes  

**Workspace name:**  
RN3_Generic_Export_Excel_v3.fmw  

**Operation:**  
Export  

**File extension/s:**  
xlsx  

**Custom parameters**  

| **Parameter key** | **Parameter value** |
| --- | --- |
| **renameTables** | *BiologyEQRClassificationProcedure,BiologyEQRClassificationProcedu* |
| **excludeTables** | *CP_reference* |

## External integration custom parameters

### Reference

**Optional parameters:**  
- **renameTables** - List of pairs of RN3 table names and matching worksheet names, where the worksheet name is different from the RN3 table name. (The parameter is available from v3 of the FME workspace.)
	```{admonition} renameTables syntax
	``RN3_tableName,WorksheetName;RN3_tableName,WorksheetName;...``
	```
- **excludeTables** - A comma-separated list of RN3 table names that should be excluded from the export. (The parameter is available from v3 of the FME workspace.)
	```{admonition} excludeTables syntax
	``RN3_tableName,RN3_tableName;...``
	```


### Explanation

- The main purpose of the **renameTables** parameter is to solve situations where the name of an RN3 table is longer than 31 characters, which is the limit for the name of an Excel worksheet. The parameter gives the export process a key for how to name a worksheet to contain data from an RN3 table with a name too long for Excel. The solution, however, can be used in any situation where the worksheet name should differ from the name of the respective RN3 table (e.g., if the exported file is to be used without additional edits in an unaligned system or process).


## Data transformations

The generic export doesn't change the RN3 data, with the following exceptions:

- Convert Date values to YYYY-MM-DD format if possible.
- Remove all Geometry values.


## FME workspace

The external integration FME workspace does the following:  
- Downloads and reads data from the RN3 dataset tables, except the data from RN3 tables set for exclusion in the **excludeTables** parameter.  
- Transforms or removes selected data values if needed and possible.  
- Writes data into an Excel file, with worksheet names matching the table names, or the names supplied in the ***renameTables** parameter.  

It downloads the data from RN3 as a ZIP file containing a set of CSV files, one per RN3 dataset table, using the asynchronous request starting with '/dataset/v4/etlExport/{datasetId}' RN3 API endpoint.  
It is not possible to specify which tables to exclude from the download; therefore, the ZIP file contains data from all tables. The 'table' exclusion is done during the data reading step.  
If the dataset contains no geometry fields, the workspace extracts the non-excluded CSV files from the ZIP file and reads them with a FME CSV FeatureReader transformer.  
If the dataset contains geometry fields, the data is read using a custom-built Python script. This is to prevent a known issue where the FME CSV Reader failed due to the size of a GeoJSON string in the file. This solution is, however, slower and has not been extensively tested for other potential issues.  

**Latest version:**  
<https://fme.discomap.eea.europa.eu/fmeserver/workspaces/run/Dataflows_RN3_Generic_Processes/RN3_Generic_Export_Excel_v3.fmw>