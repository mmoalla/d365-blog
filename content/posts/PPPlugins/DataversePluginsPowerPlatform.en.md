---
title: "Dataverse plugins - writing server-side logic for Power Platform"
date: 2026-09-22
draft: false
categories: ["Power Platform"]
tags: ["Dataverse", "Plugins", "C#", ".NET", "Power Platform CLI"]
summary: "When Power Automate or client-side scripts aren't enough, plugins let you run C# code directly in the Dataverse pipeline. Here's how they work, and how to build, register, and test a simple one."
lightbox:
  enabled: true
justified_gallery:
  enabled: true
---

# Context
 
A **plugin** is a piece of C# code that runs directly inside the Dataverse execution pipeline, triggered by a data operation - creating a record, updating a field, deleting a row. Unlike a Power Automate flow (which runs asynchronously, outside the transaction, with some delay) or a client-side script (which only runs in the browser and can be bypassed by API calls), a plugin runs **synchronously or asynchronously, server-side, inside the same database transaction** as the operation that triggered it. That makes it the right tool when you need to guarantee that a business rule is enforced no matter how the data got there — a Power Automate flow, a custom app, an import, or the standard UI.
 
# Prerequisites
 
- The [.NET SDK 4.6.2](https://dotnet.microsoft.com/fr-fr/download/dotnet-framework/net462) and the [Power Platform CLI](https://learn.microsoft.com/power-platform/developer/cli/introduction) (`pac`) installed
- A Dataverse environment you're authenticated against (`pac auth create`)
- Basic C# familiarity.

# Steps
## Create a plugin project
1. Open Visual Studio and create project.
2. In dialog box choose **Class Library (.NET Framework)** and click next. 
![](images/PPPlugins/image1.png)
2. Enter project name and choose **.NET Framework 4.6.2**
3. Click `Create` button to create project. 
![](images/PPPlugins/image2.png)

## Add Xrm sdk Core Assemblies Nuget Package
1. Right to created project and select Manage NuGet Packages.
2. Install **`Microsoft.CrmSdk.CoreAssemblies`** Nuget.
![](images/PPPlugins/image3.png)

## Write simple validation plugin
1. Create new class **PreventDuplicateAccountName** that implement **IPlugin** interface 

```C# {lineNos=true filename=PreventDuplicateAccountName}
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace PPPlugins
{
    public class PreventDuplicateAccountName : IPlugin
    {
        public void Execute(IServiceProvider serviceProvider)
        {
            IPluginExecutionContext executionContext = (IPluginExecutionContext)serviceProvider.GetService(typeof(IPluginExecutionContext));
            IOrganizationServiceFactory serviceFactory = (IOrganizationServiceFactory)serviceProvider.GetService(typeof(IOrganizationServiceFactory));
            IOrganizationService organizationService = serviceFactory.CreateOrganizationService(executionContext.UserId);

            if (executionContext.InputParameters.Contains("Target")
                && executionContext.InputParameters["Target"] is Entity entity)
            {
                if(!entity.Contains("name"))
                {
                    return;
                }

                string accountName = entity.GetAttributeValue<string>("name");

                QueryExpression query = new QueryExpression("account")
                {
                    ColumnSet = new ColumnSet("name"),
                    Criteria = new FilterExpression()
                };
                query.Criteria.AddCondition("name", ConditionOperator.Equal, accountName);
                query.Criteria.AddCondition("statecode", ConditionOperator.Equal, 0);

                EntityCollection existingAccounts = organizationService.RetrieveMultiple(query);

                if(existingAccounts.Entities.Count > 0)
                {
                    throw new InvalidPluginExecutionException($"An active account named '{accountName}' already exists.");
                }
            }
        }
    }
}
```
2. Right click to project and click **properties**.
3. Generate a certificate by signing the assembly and click save.  
4. Compile the project to generate dll file (this will be used to register assembly in power platform).
![](images/PPPlugins/image4.png)
![](images/PPPlugins/image5.png)
![](images/PPPlugins/image6.png)

## Register the plugin
1. Open Plugin Registration Tool.
2. Register new assembly.
![](images/PPPlugins/image7.png)
3. Select the dll in `bin` folder and click register selected plugin.
![](images/PPPlugins/image8.png)
4. Register a new step: message **Create**, primary entity **account**, stage **PreOperation**, execution mode **Synchronous**
![](images/PPPlugins/image9.png)
![](images/PPPlugins/image10.png)

## Test the plugin
when you try to add an existing account
![](images/PPPlugins/image11.png)

# Further reading
- [Write a plug-in — Microsoft Learn](https://learn.microsoft.com/power-apps/developer/data-platform/write-plug-in)
- [Best practices for plug-in development — Microsoft Learn](https://learn.microsoft.com/power-apps/developer/data-platform/best-practices/business-logic/)
- [Tutorial: Write and register a plug-in — Microsoft Learn](https://learn.microsoft.com/power-apps/developer/data-platform/tutorial-write-plug-in)