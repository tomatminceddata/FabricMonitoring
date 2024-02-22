For quite some time, I claimed that I’m a Microsoft Fabric/Power BI Sherpa, helping an organization (my colleagues) have a successful and safe journey (the definition of success follows in a short moment). I believe the foundation for a joyful journey is the proper configuration of the Microsoft Fabric/Power BI tenant settings (next to knowing some technical things). For this reason, I created (and still create) this solution. This solution is not final, and currently, I have no roadmap. Nevertheless, I think it’s a start.
# Tenant Setting
At the moment of this writing (February, 24th 2024), there are 119 settings. I consider the configuration of these settings foundational for successfully using Power BI and Microsoft Fabric. My definition of success:

Successfully using Microsoft Fabric and Power BI means enabling an organization to harvest insights from data without putting data assets at risk and preventing the waste of resources. Resources are the capacities.

I started this solution with these because the tenant settings are at the foundation.

According to the official documentation, tenant settings are defined as:
> Tenant settings enable fine-grained control over the 
features that are made available to your organization. If you have 
concerns around sensitive data, some of our features might not be right 
for your organization, or you might only want a particular feature to be
 available to a specific group.
# Beware
If you want to rebuild this solution on one of your Microsoft Fabric capacities, read the next two paragraphs carefully.

I can not be held responsible for any misfortune you might experience using this solution or parts of this solution (read the license document of this repo). Test this solution on a Fabric capacity that you use for development. I tried hard, but I’m only human. 

Make your assignments and add or remove risk types (adding or removing risk types requires the adaption of the dataflow). This is the Excel file in the Risktypes folder of this repo. Finally, there is a short chapter called Setup at the end of this readme, read it 😉
# Outline of the current solution
The current solution is leveraging Microsoft Fabric, comprising a data pipeline, a lake house, some notebooks, a dataflow, a semantic model, and maybe the most important piece, an Excel file.

Data ready for consumption is stored in delta tables inside a lake house. Even though I follow medallion design, data is not spread across different lake houses, workspaces, or Fabric capacities.
## Overall architecture (schematic overview)
The next image shows a schematic overview of the current current solution:
![Alt text](https://github.com/tomatminceddata/FabricMonitoring/blob/main/Images/Overall%20architecture%20-%20schematic.png)
You see this: the solution is based solely (almost) on Microsoft Fabric. The only component that is not part of Microsoft Fabric is Azure Key Vault. I use Azure Key Vault to store the secret of the Service Principal that is authenticating against the Microsoft Fabric REST API endpoint to retrieve the tenant settings. The solution is leveraging dataflows Gen 2 and notebooks for data retrieval. The execution is orchestrated by a data pipeline. The original JSON document will be stored in the files section of the lake house, I consider this useful whenever I want to rebuild the complete history. The dataflow extracts data from an Excel file and stores the data into a delta table called “fabricmonitoring_tenantsettings_settings_silver.” Because there are no transformations, the delta table will be directly stored to the silver the layer.

Note: I decided to build the medallion layer by table names because of the small amount of data and the small number of data loads (once a day).
## The Data
Currently, the data is extracted from the Microsoft Fabric Rest endpoint: https://learn.microsoft.com/en-us/rest/api/fabric/admin/tenants/get-tenant-settings?tabs=HTTP

Data extraction is done by a service principal (sometimes I imagine a service principal as my larger brother). Because a password (being precise, a secret) is required to authenticate against the REST API  endpoint, an Azure Key Vault is involved. If you have no idea about azure Key Vault, the presentation mentioned at the end contains some introductory slides.

Next to the data
## Orchestration
The orchestration of the execution of the dataflow and the notebooks is done by a single pipeline:
![Alt text](https://github.com/tomatminceddata/FabricMonitoring/blob/main/Images/FabricMonitorint_orchestration.png)
There is nothing special about the data pipeline, you will find the json of this data pipeline in the folder “Data pipeline” of this repo.
## The Excel file, assigning risk types to a setting
The Excel file contains all settings with risk types assigned. The risk types are:

- Data Residency
- Data Exfiltration
- Data Literacy
- Data Security
- Data Integrity
- Resource consumption

The idea is simple: each setting might be exposed to one or more risk types. Knowing about these risk types helps to mitigate these risks; the most drastic action to be taken is disabling a feature. Please be aware that the assignment is my point of view. 

This presentation holds a short definition of the above-mentioned risk tpyes. From a data modeling perspective, this Excel file can be considered to be the representation of the dimension table of the settings.

In my solution, this Excel file is stored in a SharePoint library and must be manually updated/adapted as soon as new settings arrive.
## The notebooks
At the current moment there are three notebooks. You will find the notebooks in the Notebooks folder of this repo. Import these notebooks to a Fabric workspace that you use for development purposes. 
### FabricMonitoring_TenantSettings_GetData
This notebook extracts the data from the REST API endpoint. Accessing the Microsoft Fabric API requires authentication. The solution is leveraging a Service Principal, a registered app in your Azure Entra Id. If you are not familiar with app registration in Azure Entra Id, I recommend you start reading here: https://learn.microsoft.com/en-us/entra/identity-platform/app-objects-and-service-principals?tabs=browser#service-principal-object

Add the app to an Entra Id group and add this group to the allowed  security groups of the setting “Service principals can use Fabric APIs.” You will find this setting in the Developer settings group
![Alt text](https://github.com/tomatminceddata/FabricMonitoring/blob/main/Images/FabricMonitoring_TenantSetting_SPNcanUseFabricAPI.png)
