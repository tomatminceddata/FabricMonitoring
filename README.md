For quite some time, I claimed that I’m a Microsoft Fabric/Power BI Sherpa, helping an organization (my colleagues) have a successful and safe journey (the definition of success follows in a short moment). I believe the foundation for a joyful journey is the proper configuration of the Microsoft Fabric/Power BI tenant settings (next to knowing some technical things). For this reason, I started with the tenant settings other aspects will follow in the next months. This solution is not done yet, and currently, I have no roadmap. Nevertheless, I think it’s a start.
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

The presentation referenced at the end of this document holds short definitions of the above-mentioned risk tpyes. From a data modeling perspective, this Excel file can be considered to be the representation of the dimension table of the settings.

In my solution, this Excel file is stored in a SharePoint library and must be manually updated/adapted as soon as new settings arrive. This Excel file will be processed by the dataflow Gen 2. You will find the Excel in the folder Risktypes of this repo.
## The notebooks
At the current moment there are three notebooks. You will find the notebooks in the Notebooks folder of this repo. Import these notebooks to a Fabric workspace that you use for development purposes. 
### FabricMonitoring_TenantSettings_GetData
This notebook extracts the data from the REST API endpoint. Accessing the Microsoft Fabric API requires authentication. The solution is leveraging a Service Principal, a registered app in your Azure Entra Id. If you are not familiar with app registration in Azure Entra Id, I recommend you start reading here: https://learn.microsoft.com/en-us/entra/identity-platform/app-objects-and-service-principals?tabs=browser#service-principal-object

Add the app to an Entra Id group and add this group to the allowed  security groups of the setting “Service principals can use Fabric APIs.” You will find this setting in the Developer settings group
![Alt text](https://github.com/tomatminceddata/FabricMonitoring/blob/main/Images/FabricMonitoring_TenantSetting_SPNcanUseFabricAPI.png)
The JSON documents are stored to the files section, the next image shows this for my environment:
![Alt text](https://github.com/tomatminceddata/FabricMonitoring/blob/main/Images/FabricMonitoring_FileFolder.png)
The relative path for the storage of the JSON documents can be configured in the JSON document called “FabricMonitoring_Variables.json.” Read about this document in the Setup chapter further down below.
### FabricMonitoring_TenantSettings_TransformData
This notebook reads the JSONs from the file section and writes the values into a delta table called “fabricmonitoring_tenantsettings_raw”

Currently this happens every time the notebook is executed. Because of the small size of the single JSON documents I consider this feasible. Another reason why I’m doing this is - the solution is in a flux and might change  over the next months.

After the “_raw” delta table is created, the merge command of the DeltaTable object is used to propagate data to the _bronze and _silver delta tables. Currently there are no _gold tables, this will change when the solution evolves.
### FabricMonitoring_TenantSettings_RefreshSemanticModel
This notebook is refreshing the semantic when executed. Refreshing of a semantic models is necessary even if the model is using the direct lake connection mode. Because of the direct lake connection mode can be considered a meta data operation. the execution takes only seconds.
## The dataflow - FabricMonitoring_TenanSettings_Settings
This dataflow reads the data from the Excel file (located in a SharePoint folder) into the the delta table “fabricmonitoring_tenantsettings_settings_silver”

At the moment this table can be considered the dimension table that represent the settings.

You will find the JSON of this dataflow in the folder “dataflows” of this repo.
## The semantic model
Currently I have no idea how to make it simple to share the metadata of a semantic model, probably I will start using Tabular Editor c# scripts in a couple of weeks. For now some screenshots and some code blocks have to be sufficient.
### The relationships of the model
Next images show the relationships of the semantic model:
![Alt text](https://github.com/tomatminceddata/FabricMonitoring/blob/main/Images/FabricMonitoring_Relationships.png)
And the properties of the relationship:
![Alt text](https://github.com/tomatminceddata/FabricMonitoring/blob/main/Images/FabricMonitoring_RelationshipProperties.png)
### The measures

- latest fileDate
latest fileDate is determined to avoid double counts
    
    ```jsx
    latest fileDate = 
    MAXX( 
        VALUES( 'fabricmonitoring_tenantsettings_silver'[fileDate] )
        , 'fabricmonitoring_tenantsettings_silver'[fileDate]
    )
    ```
    
- “# no of settings” (home table)
Counts the no of settings, because at the current moment there is. no real fact table, DISTINCTCOUNT is used
    
    ```jsx
    # of Settings = 
    var latestFileDate = [latest fileDate]
    return
    CALCULATE( 
        DISTINCTCOUNT( 'fabricmonitoring_tenantsettings_settings_silver'[settingName] ) 
        , 'fabricmonitoring_tenantsettings_silver'[fileDate] = latestFileDate
    )
    ```
    
- state
this measure determines the state of a setting,
    - -1, this is the state for the first measurement
    - 0, the setting is not new
    - 1, the setting never appeared before, a new setting
    
    ```jsx
    state = 
    var previousDate = 
        MAXX(
            OFFSET(
                -1
                ,SUMMARIZE(
                    ALLSELECTED( 'fabricmonitoring_tenantsettings_silver'[fileDate] )
                    , 'fabricmonitoring_tenantsettings_silver'[fileDate]
                )
                , ORDERBY( 'fabricmonitoring_tenantsettings_silver'[fileDate] , ASC)
                , DEFAULT
            )
            , 'fabricmonitoring_tenantsettings_silver'[fileDate]
        )
    var isknown =
        CALCULATE( LASTNONBLANK( 'fabricmonitoring_tenantsettings_settings_silver'[settingName] , 0) , 'fabricmonitoring_tenantsettings_silver'[fileDate] = previousDate)
    return
    IF( ISBLANK( previousDate )
        , -1
        , IF( NOT( ISBLANK( isknown ) ), 0 , 1)
    )
    ```
# Why I’m doing this´
If you are wondering why I’m doing this, on February 22nd, 2024, a new setting appeared. All the above helps to spot new settings easily, and this is only the most obvious use case.
A simple visualization that shows the new setting:
![Alt text](https://github.com/tomatminceddata/FabricMonitoring/blob/main/Images/FabricMonitoring_ANewSetting.png)
# Setup this solution
## Create a lakehouse (or use an existing one)

The name of a lake house is required for the configuration file (see the next section); in addition, the result of an API request will be stored as a JSON file in the files section (currently, the solution is using a relative path; this might change in the future.

## The configuration file - FabricMonitoring_Variables.json

You will find this file in the folder “GeneralVariables” of this repo. Currently, it is mandatory to upload this JSON document to the root files folder of the default lake house, meaning the lake house you assign to the notebooks you will find in this repo. Currently it’s also mandatory to assign a lakehouse to the notebooks.

The variables that can be configured:

- tenantId
The tenant Id that is used to authenticate against Microsoft’s Fabric REST API endpoint, , adapt the value accordingly
- clientId
The client Id of the Service Principal used to authenticate against Microsoft’s Fabric REST API endpoint, adapt the value accordingly. Basically this is the user name, a service principal can also be considered being a user.
- nameOfTheKeyVault
The name of the Azure Key Vault that holds the secret of the SPN, adapt the value accordingly.
- nameOfTheKey
The name of the secret, adapt the value accordingly. This is just a name to identify the secret.
- filepathTenantSettings
The relative path where the JSON documents will be stored, adapt the path accordingly
- theLakehouse
The name of the lake house, adapt the value accordingly. This name is used to determine if delta tables have to be created or not.
- deltaTablePrefix
I use a prefix for deltatables, to separate tables in the lakehouse by “solution.” I do this because I consider it more manageable having one lake house instead of having multiple lakehouse for every solution. A table name will look like this:
fabricmonitoring_tenantsettings_bronze
Adapt the value accordingly

```
"tenantId": "put your tenantid here",
"clientId":  "put the client id of the SPN here",
"nameOfTheKeyVAult": "put the name of the Azure Key Vault here",
"nameOfTheKey": "put the name of the secret here",
"filepathTenantSettings": "Files/FabricMonitoring/TenantSettings/",
"theLakehouse": "theLakehouse",
"deltaTablePrefix": "fabricmonitoring_"
```
## Import the notebooks

Import the notebooks into the workspace you want to use for this solution. Assign the lakehouse you want to use as the default.

## Create the folder structure

Create the folder structure you configured in the configuration file.
## Run the notebooks
Start with
1 FabricMonitoring_TenantSettings_GetData
2 FabricMonitoring_TenantSettings_TransformData
3 FabricMonitoring_TenantSettings_RefreshSemanticModel (this step is only necessary, if you created custom semantic model)
# Some additional notes
This is the link to a presentation at the Global Power Platform Bootcamp Hamburg. This presentation contains definitions of the risk types: https://github.com/tomatminceddata/TomsPublicSpeaking/blob/main/20240224%20-%20GPPBCHH%20-%20Tom%20Martens%20-%20Microsoft%20Fabric%20Tenant%20Settings/20240224%20-%20GPPBC%20-%20Tom%20Martens%20-%20Microsoft%20Fabric%20Tenant%20Settings%20-%2020240224.pptx
