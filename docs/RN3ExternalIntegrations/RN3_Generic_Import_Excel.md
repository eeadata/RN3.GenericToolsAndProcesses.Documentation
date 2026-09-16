# RN3 Generic Import - Excel

___

**Complexity:**  
<span class="stars">⯪☆☆☆☆</span>

**Operation type:**  
IMPORT  

## Introduction

This external integration allows an authorised data reporter user to import data from an MS Excel file into an RN3 dataset.  

The structure of the Excel file should match the structure of the RN3 dataset.  
- The names of worksheets in the file should match the names of the RN3 tables. For exceptions see **'renameTables'** in the {ref}`RN3_Generic_Import_Excel.md-external-integration-custom-parameters`. The process ignores letter case differences between the worksheet and RN3 table names.
- The names of the worksheet columns must match the names of the fields in the corresponding RN3 tables, but they don't need to follow the same letter case.  
- Worksheets with non-matching names will be ignored. RN3 tables with missing worksheets will not be updated.  
- Additional or misnamed worksheet columns will be ignored. RN3 fields with missing worksheet columns will be empty after the import.  

```{important}
If an RN3 table name is longer than the maximum allowed length of an Excel worksheet name (31 characters), add the **'renameTables'** parameter to the external integration setup, where you provide the mapping key between the actual worksheet name and the corresponding RN3 table name.  

*(The renameTable parameter is available from v3 of the FME workspace.)*  
```

```{note}
The data provider decides whether they want to replace existing data in the initial Import dialogue window (the Replace data checkbox).  
```

```{warning}
If the data provider chooses to replace the data, all existing data across all dataset tables will be deleted immediately. If the import fails for whatever reason, the dataset tables will remain empty.
```

```{note}
With some exceptions, the process doesn't prevent import of values that do not match the constraints of the dataset table fields (e.g., data types). The discrepancies will be identified during the RN3 Validation.  
See {ref}`RN3_Generic_Import_Excel.md-data-transformations` for details on the exceptions.
```

```{caution}
Excel file is not a suitable format for importing **geometry data**. A different external integration should be used for that purpose. 
```


## External integration setup - example

**Name:**  
Import Excel file  

**Description:**  
Import Excel file (generic)  

**Repository:**  
Dataflows_RN3_Generic_Processes  

**Workspace name:**  
RN3_Generic_Import_Excel_v3.fmw

**Operation:**  
IMPORT

**File extension/s:**  
xls,xlsx 

**Custom parameters**  

| **Parameter key** | **Parameter value** |
| --- | --- |
| **flexiDateFields** | *eionetChangeDate* |
| **renameTables** | *BiologyEQRClassificationProcedu,BiologyEQRClassificationProcedure* |

(RN3_Generic_Import_Excel.md-external-integration-custom-parameters)=
## External integration custom parameters

### Reference

**Optional parameters:**  
- **flexiDateFields** - Comma-separated list of 'flexible' date field names.
- **renameTables** - A mapping key between the worksheet names and matching RN3 table names, where the two names are different. (The parameter is available from v3 of the FME workspace).
	```{admonition} renameTables syntax
	``WorksheetName,RN3_tableName;WorksheetName,RN3_tableName;...``
	```


### Explanation

- The 'flexible' date fields are RN3 fields, which are of Date type (YYYY-MM-DD format), but it's acceptable if a data provider reports only year (YYYY) or only year and month (YYYY-MM) values in the corresponding Excel worksheet fields. The import procedures convert such values to the full date format by adding '-01' to fill in the missing parts.  
- The main purpose of the **renameTables** parameter is to solve situations where the name of an RN3 table is longer than 31 characters, which is the limit for the name of an Excel worksheet. The parameter thus gives the import process a key for mapping the shorter worksheet name to the longer RN3 table name, ensuring the worksheet data ends in the correct table. The solution, however, can be used in any situation where the worksheet name differs from the name of the corresponding RN3 table (e.g., if the data providers obtain the Excel files from an unaligned system and their editing is not feasible, allowed or recommended).  


(RN3_Generic_Import_Excel.md-data-transformations)=
## Data transformations

The infamous 'smartness' of MS Excel's value formatting can cause values to be read and imported incorrectly.  
The file may also contain empty records, cells with empty characters, values consisting fully of whitespace characters, or values with leading or trailing spaces.  
Imported data with these issues often fail a relevant QC test during the RN3 dataset Validation, but it may be difficult to see why at first glance.  

To avoid the most common of these issues, to make the data provider's life easier, and to reduce the load on the dataflow helpdesk at the same time, the generic import does the following content and value transformations:

- Trim the leading and trailing spaces from values, and replace values consisting only of whitespace characters with null.
- Remove records with no values.
- Convert Date values to YYYY-MM-DD format if possible.
- Convert Datetime values to YYYY-MM-DD hh:mm:ss format if possible.
- Convert values in scientific E-notation to proper decimal or integer values.
- Remove trailing decimal zeroes from integer values
- Identify RN3 fields with (pseudo-)boolean codelists, and convert values from the respective Excel file fields to the corresponding codelist values, if possible.
	
	```{admonition} Example
	If an RN3 field has a codelist with items 'true' and 'false', the value 'Yes' reported in the corresponding Excel file field will be converted to 'true'.
	```
	
	- The following value pairs are considered (pseudo-)boolean for this transformation:
		- 0, 1  
		- true, false  
		- yes, no  
	- The procedure is case-insensitive (i.e., it recognizes and handles values with all possible case variations. E.g., 'true', 'TRUE', 'True', 'tRuE', ...).
	- Only RN3 fields with codelists ('Single select' or 'Multiple select' type) are handled by this procedure. The 'Link' type fields are not, even if the linked reference table contains (pseudo-)boolean values.
	- If the RN3 field codelist has more than two items, the field is excluded from this procedure, even if some of the items appear (pseudo-)boolean.
	


## FME workspace

The external integration FME workspace does the following:  
- Reads data from the supplied Excel file.  
- Change the letter case of the worksheet name attributes if they differ from the RN3 table names.  
- Change the letter case of the column attribute names if they differ from the RN3 field names.  
- Transforms selected data values if needed and possible.  
- Imports the result into the dataset.  

It transforms and saves the data into a series of CSV files (one per valid worksheet table), zips the CSV files into one ZIP file, which it then imports to the RN3 reporting dataset using the '/dataset/v2/importFileData/{datasetId}' RN3 API endpoint.  

**Latest version:**  
<https://fme.discomap.eea.europa.eu/fmeserver/workspaces/run/Dataflows_RN3_Generic_Processes/RN3_Generic_Import_Excel_v3.fmw>