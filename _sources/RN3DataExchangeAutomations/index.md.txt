(rn3-data-exchange-automations)=
# RN3 Data exchange automations

```{toctree}
:maxdepth: 2
:caption: Table of Contents
:hidden:

RN3_Generic_Harvesting_Metadata
RN3_Generic_Harvesting_Data
RN3_Generic_Update_Reference

```

Most dataflows include exchange of data between the RN3 platform and a dedicated data processing or storage platform, for example, harvesting of reported data or updates of specific data in the RN3 dataflow.

On a small scale, these actions can be done manually using the native RN3 export and import, or external integrations. These options become inadequate when we need to do them regularly, frequently, and on a large scale.

The RN3 API service and FME let us automate and scale up these processes.
Generic FME based solutions have been designed for the automation of the following dataflow processes:

- Dataflow Metadata harvesting
- Dataflow Data harvesting
- Dataflow Reference data update

These processes include multiple steps, involve multiple components, and generally require more time and effort than setting up external integrations. The documentation for each solution provides detailed descriptions of the components and includes a **Quick setup** section that outlines the necessary steps, their order and references to the respective parts of the documentation.

## Notifications

If the automated processes are set up to run regularly, it is recommended to also set up notifications to inform responsible actors about the results of individual FME jobs and potential failures or other problems.

```{seealso}
<https://docs.safe.com/fme/html/FME-Flow/WebUI/notifications.htm>
```