# RN3 Generic Import - Excel

___

**Description:**  
This external integration adds an **Import dataset data** option to the RN3 dataset, allowing the data provider to import data in MS Excel format.  

The names of the worksheets in the Excel file must correspond to the names of the tables in the RN3 dataset. Worksheets with non-matching names will be ignored. RN3 tables with missing worksheets will not be updated.

```{warning}
If an RN3 table name is longer than the maximum allowed length of an Excel worksheet name, such a table can't be imported using this generic external integration.
```

```{note}
The data provider decides whether they want to replace existing data in the initial Import dialogue window (the Replace data checkbox).  
```

```{warning}
If the data provider chooses to replace the data, all existing data across all dataset tables will be deleted immediately. If the import fails for whatever reason, the dataset tables will remain empty.
```

```{warning}
With some exceptions, the process doesn't prevent import of values that do not match the constraints of the dataset table fields (e.g., data type). The discrepancies will be identified during the RN3 Validation.  
See {ref}`RN3_Generic_Import_Excel.md-data-transformations` for details on the exceptions.
```


**Operation type:**  
IMPORT  


## External integration setup - example

**Name:**  
Import Excel file  

**Description:**  
Import Excel file (generic)  

**Repository:**  
Dataflows_RN3_Generic_Processes  

**Workspace name:**  
RN3_Generic_Import_Excel.fmw

**Operation:**  
IMPORT

**File extension/s:**  
xls,xlsx 

**Custom parameters**  

| **Parameter key** | **Parameter value** |
| --- | --- |
| **flexiDateFields** | *eionetChangeDate* |


## External integration custom parameters

### Reference

**Optional parameters:**  
- **flexiDateFields** - Comma-separated list of 'flexible' date field names.


### Explanation

- The 'flexible' date fields are RN3 fields, which are of Date type (YYYY-MM-DD format), but it's acceptable if a data provider reports only year (YYYY) or only year and month (YYYY-MM) values in the corresponding Excel worksheet fields. The import procedures convert such values to the full date format by adding '-01' to fill in the missing parts.  

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
- Transforms selected data values if needed and possible.  
- Imports the result into the dataset.  

It transforms and saves the data into a series of CSV files (one per valid worksheet table), zips the CSV files into one ZIP file, which it then imports to the RN3 reporting dataset using the '/dataset/v2/importFileData/{datasetId}' RN3 API endpoint.  

**Latest version:**  
<https://fme.discomap.eea.europa.eu/fmeserver/workspaces/run/Dataflows_RN3_Generic_Processes/RN3_Generic_Import_Excel.fmw>