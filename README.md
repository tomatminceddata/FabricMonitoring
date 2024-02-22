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
