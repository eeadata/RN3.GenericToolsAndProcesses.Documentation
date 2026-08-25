---
html_theme.sidebar_secondary.remove: true
---

# RN3 Generic Tools and Processes - Documentation

```{toctree}
:maxdepth: 2
:caption: Contents:
:hidden:

RN3ExternalIntegrations/index
RN3Harvesting/index
SupportingTools/index

```

RN3 Generic Tools and Processes are predominantly FME-based solutions, providing added functionality to actors involved in Reportnet 3 dataflows.  

In this documentation, you will find information on how to set-up and use these tools an processes.

```{note}
The documentation is intended for Data custodians and their consultants, or Data stewards confident in their technical skills (see {ref}`gtaps-index.md-requirements` for more details).
```


## Categories

Main categories are:  
- {ref}`rn3-external-integrations`
- {ref}`rn3-harvesting-procedures`  
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
