---
title: "Plugins Dataverse - écrire une logique côté serveur pour Power Platform"
date: 2026-09-22
draft: false
categories: ["Power Platform"]
tags: ["Dataverse", "Plugins", "C#", ".NET", "Power Platform CLI"]
summary: "Lorsque Power Automate ou les scripts côté client ne suffisent pas, les plugins permettent d'exécuter directement du code C# dans le pipeline Dataverse. Voici comment ils fonctionnent et comment créer, enregistrer et tester un plugin simple."
lightbox:
  enabled: true
justified_gallery:
  enabled: true
---

# Contexte

Un **plugin** est un morceau de code C# qui s'exécute directement dans le pipeline d'exécution Dataverse, déclenché par une opération sur les données : création d'un enregistrement, mise à jour d'un champ ou suppression d'une ligne. Contrairement à un flux Power Automate (qui s'exécute de manière asynchrone, en dehors de la transaction et avec un certain délai) ou à un script côté client (qui ne s'exécute que dans le navigateur et peut être contourné par des appels d'API), un plugin s'exécute **de manière synchrone ou asynchrone, côté serveur, dans la même transaction de base de données** que l'opération qui l'a déclenché. C'est donc l'outil adapté lorsque vous devez garantir l'application d'une règle métier, quelle que soit l'origine des données : un flux Power Automate, une application personnalisée, une importation ou l'interface utilisateur standard.

# Prérequis

- Le [SDK .NET 4.6.2](https://dotnet.microsoft.com/fr-fr/download/dotnet-framework/net462).
- Une connaissance de base de C#.

# Étapes
## Créer un projet de plugin
1. Ouvrez Visual Studio et créez un projet.
2. Dans la boîte de dialogue, choisissez **Class Library (.NET Framework)**, puis cliquez sur Suivant.
![](images/PPPlugins/image1.png)
2. Saisissez le nom du projet et choisissez **.NET Framework 4.6.2**.
3. Cliquez sur le bouton `Create` pour créer le projet.
![](images/PPPlugins/image2.png)

## Ajouter le package NuGet Xrm SDK Core Assemblies
1. Faites un clic droit sur le projet créé et sélectionnez Manage NuGet Packages.
2. Installez le package NuGet **`Microsoft.CrmSdk.CoreAssemblies`**.
![](images/PPPlugins/image3.png)

## Écrire un plugin de validation simple
1. Créez une nouvelle classe **PreventDuplicateAccountName** qui implémente l'interface **IPlugin**.

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
2. Faites un clic droit sur le projet et cliquez sur **Properties**.
3. Générez un certificat en signant l'assembly, puis cliquez sur Enregistrer.
4. Compilez le projet pour générer le fichier DLL (il sera utilisé pour enregistrer l'assembly dans Power Platform).
![](images/PPPlugins/image4.png)
![](images/PPPlugins/image5.png)
![](images/PPPlugins/image6.png)

## Enregistrer le plugin
1. Ouvrez le Plugin Registration Tool.
2. Enregistrez une nouvelle assembly.
![](images/PPPlugins/image7.png)
3. Sélectionnez la DLL dans le dossier `bin`, puis cliquez sur Register Selected Plugin.
![](images/PPPlugins/image8.png)
4. Enregistrez une nouvelle étape : message **Create**, entité principale **account**, étape **PreOperation**, mode d'exécution **Synchronous**.
![](images/PPPlugins/image9.png)
![](images/PPPlugins/image10.png)

## Tester le plugin
Lorsque vous essayez d'ajouter un compte existant :
![](images/PPPlugins/image11.png)

# Pour aller plus loin
- [Écrire un plug-in — Microsoft Learn](https://learn.microsoft.com/power-apps/developer/data-platform/write-plug-in)
- [Bonnes pratiques pour le développement de plug-ins — Microsoft Learn](https://learn.microsoft.com/power-apps/developer/data-platform/best-practices/business-logic/)
- [Tutoriel : écrire et enregistrer un plug-in — Microsoft Learn](https://learn.microsoft.com/power-apps/developer/data-platform/tutorial-write-plug-in)
