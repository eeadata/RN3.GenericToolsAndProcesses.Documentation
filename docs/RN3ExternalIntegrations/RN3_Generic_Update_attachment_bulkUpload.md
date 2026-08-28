# RN3 Generic Update - attachment bulk upload

___

**Complexity:**  
<span class="stars">★☆☆☆☆</span>

**Description:**  
This external integration adds an **Import dataset data** option to the RN3 dataset.  
It allows the data provider to bulk upload attachment files into a single RN3 reporting dataset table (***attachment table***).  

The main use case is RN3 datasets where the data provider may need to upload a large number of attachment files, which they would otherwise need to upload one by one to the ***attachment field*** of each record in the *attachment table*.  
Instead of uploading files to the RN3, the providers upload them to a publicly accessible location. They then add download URLs of these files to a dedicated ***url field*** of the corresponding records in the *attachment table* (or use one of the import options to import the whole table).  
The external integration process attempts to us the supplied URLs and download the files. If successful, it uploads them to the *attachment field* of the records in the *attachment table*.  

To give the data provider information on how successful the upload job was, or why it failed, the process can be configured to import job summary log into a field (***log field***) in a dedicated dataset table (***log table***).

```{warning}
Any existing attachments will be removed from the table during the upload process!  
```

```{important}
The data providers must supply download URLs that point directly to the publicly accessible files!  
The files must have one of the allowed file extensions and be within the specified size limit!  
URLs and files that fail this criteria will not be used and the attachment field of the corresponding record will be left empty.
```

```{note}
Data providers can use the same download URL in multiple records. The same file will be attached to each of the records.  
```

**Operation type:**  
IMPORT FROM OTHER SYSTEM  

## Dataset design

**Basics**  
For the external integration to work, the RN3 reporting dataset must include:  
- an RN3 reporting dataset table (***attachment table***) with an Attachment type field (***attachment field***)  
- a URL type field in the *attachment table* where the data provider supplies URLs of the files to be downloaded and used as attachments (***url field***)  

Optionally, if data providers should be able to see the upload job summary log:  
- an RN3 reporting dataset table (***log table***) with a Multiline text field (***log field***) to store the summary.  

**Other considerations**  
- The *attachment table* and the *log table* must be located in the same dataset.
- If the reporting dataset contains multiple *attachment tables* where the bulk attachment upload should be implemented, each needs its own external integration.  
- The *attachment table* should contain only a single Attachment type field. If multiple attachment fields are present, the *attachment field* that is not the subject of the external integration would be emptied in the process.
- The *attachment table* may contain additional fields beside the *attachment* and *url*, but it must not containg any **geometry type fields**!  
- The *log field* must be the only field in the *log table*!  
- The upload process doesn't delete existing *log table* data. The log text is inserted to the table as a new record. This means the table can be used as a *log table* for multiple bulk upload (or other) external integrations.  
- If the reporting dataset contains any additional tables besides the *attachment table* and the *log table*, they will not be affected by the upload process.  

```{tip}
:class: dropdown, toggle-shown
- If the RN3 dataflow contains multiple tables in multiple datasets where attachment files may be included besides other data, or there's a high possibility that the data providers attach the same file to multiple records, it may be advantageous to rather collect all attachment files in a single dedicated table in a separate reporting dataset schema.  
- The attachment table, besides the *attachment* field, *url* field, and other fields like, for example, description, will also contain a custom UID field, assigned as the Primary key.  
- Data providers give each attachment record its own UID.  
- The attachment fields in the other tables are then replaced with a LINK field linked to the UID field in the attachment table (and potentially with the description field as the label), thus serving as the foreign key, and assuring consistency of the information.  
- This way the data providers need to upload each file, or its URL, only once.
```

## External integration setup - example  

**Name:**  
AdditionalDocuments - attachment upload from URL  

**Description:**  
AdditionalDocuments - attachment upload from URL (generic)  

**Repository:**  
Dataflows_RN3_Generic_Processes  

**Workspace name:**  
RN3_Generic_AttachmentBulkUpload.fmw  

**Operation:**  
IMPORT FROM OTHER SYSTEM  

**Custom parameters**  

| **Parameter key** | **Parameter value** |
| --- | --- |
| **attTableName** | *AdditionalDocuments* |
| **attFieldNameFile** | *document* |
| **attFieldNameUrl** | *fileURL* |
| **logTableName** | *AttachmentUploadLog* |
| **logFieldName** | *log* |


## External integration custom parameters

### Reference

**Mandatory parameters:**  
- **attTableName** - Name of the RN3 dataset table that contains an *attachment field* (***attachment table***).  
- **attFieldNameFile** - Name of the Attachment type field in the *attachment table* (***attachment field***).
- **attFieldNameUrl** - Name of the URL type field in the *attachment table* where data providers will import the URLs of the files to be downloaded and uploaded to the *attachment field* (***url field***).

**Optional parameters:**  
- **logTableName** - Name of the RN3 dataset table that contains a *log field* (***log table***).  
- **logFieldName** - Name of the Multiline text type field in the *log table* to store the bulk upload job summary log text (***log field***).  


## FME workspace

The external integration FME workspace does the following:  
- Checks validity of the supplied user parameters.  
- Downloads the *attachment table* data.  
- Extracts URLs from the *attachment url field*.  
- Checks if the URLs are valid and lead to files that match the *attachment file field* criteria (extension, size).  
- Downloads the valid files.  
- Creates an attachment table package with the attachment files and updated *attachment table* content, and imports it to the reporting dataset.  
- Optionally, creates an upload job summary log and imports it to a specified log table.  

The process downloads the *attachment table* data as a CSV file, using the /dataset/v4/etlExport/{datasetId} RN3 API endpoint:  

The updated *attachment table* data is saved as a CSV file and then zipped together with the attachment files within a prescribed folder structure.  
The zip package is imported through a multistage procedure utilising the following RN3 API endpoints:  
- /dataset/{datasetId}/generateImportPresignedUrl  
- /dataset/{datasetId}/etlImportDL  

The log data is saved as a csv file and imported to the dataset using the /dataset/v2/importFileData/{datasetId} RN3 API enpoint.  

**Latest version:**  
<https://fme.discomap.eea.europa.eu/fmeserver/workspaces/run/Dataflows_RN3_Generic_Processes/RN3_Generic_AttachmentBulkUpload.fmw>
