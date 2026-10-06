---
html_theme.sidebar_secondary.remove: true
---

# RN3 Generic Tools and Processes - Documentation

```{toctree}
:maxdepth: 2
:caption: Table of Contents
:hidden:

RN3ExternalIntegrations/index
RN3DataExchangeAutomations/index
SupportingTools/index

```

RN3 Generic Tools and Processes are predominantly FME-based solutions, providing added functionality to actors involved in Reportnet 3 dataflows.  

In this documentation, you will find information on how to set up and use these tools and processes.

```{note}
The documentation is intended for Data custodians and their consultants, or Data stewards confident in their technical skills (see {ref}`gtaps-index.md-requirements` for more details).
```

The main reason for choosing a generic tool or procedure is to save resources that would otherwise be spent on developing and maintaining custom solutions.  

While generic solutions can't cover all possible situations, they cover the most common ones and often provide sufficient customisation options.  

By using them, dataflow managers are also steered towards standard approaches to the design and handling of dataflows and related workflows, which may translate into additional resource savings.  

```{important}
All documented tools and processes were designed and tested on Dataflows created in the DLH version of the Reportnet 3 platform.  
They should not be used in dataflows on the original PostgreSQL version of the platform.
```

## Categories

Main categories are:  
- {ref}`rn3-external-integrations`
- {ref}`rn3-data-exchange-automations`  
- {ref}`rn3-supporting-tools`

(gtaps-index.md-requirements)=
## Complexity and technical requirements

Individual tools and processes vary in complexity and the level of technical skill required.  

### Repornet 3

Every tool, in general, requires sufficient understanding of Reportnet 3, its functions and mechanisms, and knowledge of RN3 Dataflow design principles.  

The simplest RN3 External integrations do not require additional knowledge besides how to create them.  

### SQL

More complex tools use MS SQL databases, either as a source of data or metadata, or as a place to store them.  
These tools often include SQL scripts (or their examples) for creating database objects and for managing data.  
The user setting up or using the respective tool needs to know how to use SQL Server Management Studio to modify and execute these scripts, or how to design some themselves.  
However, the creation of the SQL databases must be done by the EEA Service desk.  

### FME

Some knowledge of how to use FME Flow is required to set up RN3 Harvesting procedures (creation of Schedules or Automations) and for some RN3 External integrations using the MS SQL database (creation of database connections).
Some of these actions may, however, be outsourced to EEA Service desk. 

Knowledge of FME Form is required for tools executed locally. This includes the use of many of the Supporting tools, or testing and the potential modification of other tools and processes. In certain cases, this may also require knowledge of Python scripts.

### Other

Some tools use additional files, such as Excel or CSV files, as data sources or outputs.  


## Custom vs Generic

The custom FME solutions are created for a specific Reportnet 3 dataflow, dataset or table. This specificity affects how FME workspaces are designed, particularly regarding the schema of the inputs and outputs. For example:  
- **Readers/FeatureReaders** expect an input to have a static schema, and will always output an exact set of exposed attributes.  
- **Writers/FeatureWriters** output exposed attributes into an automatically or manually defined fixed schema.  
- **SQLCreators/SQLExecutors** contain SQL expressions with names of tables and columns hardcoded.  
- The **database connections** used by the Readers, Writers and SQLCreators/Executors are fixed.  
- **HTTPCallers** use many hardcoded parameters and sometimes even hardcoded authorisation attributes (which may be a security risk).  
- The other transformers work only with exposed attributes of fixed names.  

While the custom FME workspace may be easier to design initially, any change to the inputs or outputs may require extensive redesign.   
Inputs and outputs with different structures may require:  
1. Forks in the workspace workflow, where each path may contain copies of the same set of transformers, just with a different set of exposed attributes.  
2. Separate FME workspaces.  

Design and maintenance of custom solutions is therefore often very resource-demanding.

With some limitations, the generic solutions are designed to work independently of the schema of the workspace inputs and outputs. That allows them to be used for any input of the same general origin (e.g., an Excel file based on an official reporting template) and any output of the same type (e.g., a set of CSV files ready for import into an RN3 dataset).
Here's the list of the most common techniques that allow such an approach.
- **FeatureReaders** and **SQLExecutors** do not expose the data attributes in their outputs.  
- The schema of the input and output is inferred from the Data Schema output of the Reader transformer, or from an external source (e.g., schema of the respective RN3 dataset acquired by an HTTP API request). This schema may be used directly, but more often it's parsed and modified to create a dynamic schema for the specific output format.  
- **FeatureWriters** use a Dynamic schema definition for the creation of their outputs.
- **SQLExecutors** use values of the input attributes to construct their SQL expressions dynamically.
- The names of **database connections** are supplied dynamically as attribute values rather than fixed.
- The majority of parameters for **HTTPCallers** are supplied dynamically as attribute values.
- The **Dereferencer** transformers are used where a value from a specific unexposed attribute needs to be used or transformed.
- The **PythonCaller** transformers are used to modify or create values of unexposed attributes, or apply certain logic to the data, based on the values in unexposed attributes.
- The parameters that the FME workspace needs to work for a specific RN3 dataflow, dataset or table are supplied directly as user parameters by the RN3 platform itself (e.g., RN3 dataset identifier), from the custom parameters of the specific external integration, from the workspace execution schedule, or manually by the user when executing the workspace directly, or they are provided in an external source (e.g., dedicated SQL table). The workspace will often access and read parameters from this external source using the specific values provided in the supplied user parameters (e.g., the name of the dedicated SQL table, its schema, the database name, and the name of the corresponding FME database connection).

Designing a generic solution is more challenging, and so may be setting them up. They are, however, more efficient in the long run. One workspace can serve many different dataflows. Any corrections and updates must be made in only one workspace.  

The data managers who use custom solutions may need to update them all one by one every time RN3 behaviour changes or a new function is introduced.  
If they use the generic solutions, and the change has been implemented in the workspace version they already use, they will not need to take any action.  

If a new version of the FME workspace has been created, they can decide whether to switch after consulting the update documentation. Even then, all they may need to do in their external integration setup or FME Flow schedule is to update the workspace reference to a new version, and they can be confident the process will still work.  