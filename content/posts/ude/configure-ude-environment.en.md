---
title: "Create and configure Unified Development Environment"
date: 2026-08-12
draft: false
weight: 1
categories: ["Admin & Power Platform"]
tags: ["UDE", "Power Platform", "Platform Admin Center", "Dataverse", "Environnements"]
summary: "Microsoft is gradually replacing Cloud Hosted Environments (CHE) managed through LCS with UDE, a development environment natively connected to the Power Platform. Here's how to set it up."
---

## Context
For years, developing on Dynamics 365 F&O meant provisioning a **Cloud Hosted Environment (CHE)** from Lifecycle Services (LCS): a dedicated VM that had to be patched, maintained, and that often kept incurring storage costs even after being shut down.
 
Microsoft introduced a new approach: the **Unified Developer Experience (UDE)**. The core idea is to run finance and operations apps as an application hosted by **Microsoft Dataverse**, alongside Power Apps or Power Automate — rather than as an isolated ERP system requiring its own infrastructure.
 
Concretely, with UDE:
- Code editing and debugging still happen locally, in Visual Studio.
- But **code execution happens in the cloud** — no more VHDs to manage, no VM to keep running.
- Provisioning and administration go through the **Power Platform Admin Center (PPAC)** instead of LCS.
- Deployment now follows a pipeline-based approach (Azure DevOps), integrated with the broader Power Platform ecosystem
| | CHE (previous model) | UDE (new model) |
|---|---|---|
| Provisioning | LCS | Power Platform Admin Center |
| Infrastructure | Dedicated VM to manage | Managed by Microsoft |
| Code execution | On the VM | In the cloud |
| Cost when idle | Storage can keep billing | Optimized, no VHD to store |
| Power Platform integration | Weak / manual | Native (Dataverse, Power Apps, Power Automate) |

## Prerequisites
- As a prerequisite, you need access to a provisioned developer-focused sandbox environment.
- The user account you'll be using for development in the sandbox environment must be assigned the System Administrator role.
- The development machine should have at least 16 GB of free space on the local system drive to download the extension and metadata.
- The development machine running Microsoft Windows 10 or 11 must have Visual Studio 2022 installed with at least the .NET desktop development workload, the Modeling SDK and few other components.
- This SDK and other components can be selected and installed from the individual components pane in the Visual Studio installer. Refer to required Visual Studio components.
- Microsoft SQL Server Express LocalDB is installed by default with Visual Studio 2022, and you should validate that you can connect to it with windows authentication.

## Known limitations
- The environment name can't exceed 20 characters. This limitaion applies to finance and operations runtime.
- When installing the **Dynamics 365 Finance and Operations Provisioning App** through the Power Platform admin center on an existing organization, you may encounter an error if the organization is an unsupported Azure region within a region.
*The error says, The selected region dos not support the FnO app deployment.* To avoid this error, you can request Microsoft to move the environment to a supported region via support ticket, or provision a new environment in a different supported region.

## Steps
### 1. Provision or pick an environment in the PPAC
1. Sign in to the [Power Platform Admin Center](https://admin.powerplatform.microsoft.com).
2. Go to **Environments** and click new button.
{{< img src="images/ude/image1.png" alt="Create new environment" >}}
Make sure you enter a Unique Name, choose type **Sandbox** and Enable **Add a Dataverse data store**
3. Enable Dynamic 365 app on the next screen.
{{< img src="images/ude/image2.png" alt="Enable Dynamic 365 app on the next screen" >}}
4. Once the Environment is ready, open its details screen and select Dynamics 365 apps in resources on the top and install.**Dynamics 365 Finance and Operations Platform Tools**
5. Once that’s done we need to install another app **Finance and Operations Provisioning app.**
Select it, click next and you will be taken to a new page:
{{< img src="images/ude/image3.png" alt="Enable Dynamic 365 app on the next screen" >}}
Select the **Enable Developer Tools for Finance and Operations** and **Enable Demo Data for Finance and Operations** checkboxes. and select the appropriate version of the product and click on Install. 
{{< img src="images/ude/image4.png" alt="Enable Dynamic 365 app on the next screen" >}}
After installation if you see this card in PP environment, it confirms that the environment Is UDE and you can see F&O URL.
{{< img src="images/ude/image5.png" alt="Enable Dynamic 365 app on the next screen" >}}
{{< img src="images/ude/image6.png" alt="Enable Dynamic 365 app on the next screen" >}}
5. Go to visual studio, select Extensions > Manage Extensions.
6. Install the Power Platform extension for Visual Studio.
{{< img src="images/ude/image7.png" alt="Enable Dynamic 365 app on the next screen" >}}
7. Open Visual Studio again, you should see some new items under the Tools menu.
{{< img src="images/ude/image8.png" alt="Enable Dynamic 365 app on the next screen" >}}
8. Click on Connect To Dataverse, Check sign in as current user and Display list of available organization. On the displayed list choose your Environment.
{{< img src="images/ude/image9.png" alt="Enable Dynamic 365 app on the next screen" >}}
9. Choose the UDE environment that was previously created and click login.
{{< img src="images/ude/image10.png" alt="Enable Dynamic 365 app on the next screen" >}}
10. Waiting for Visual Studio to connect to Dataverse environment and select a solution **(this is not important choose default)**.
{{< img src="images/ude/image11.png" alt="Enable Dynamic 365 app on the next screen" >}}
Once it’s connected it will ask you if you want to download metadata, the F&O extension for VS and other assets, click Yes.
{{< img src="images/ude/image12.png" alt="Enable Dynamic 365 app on the next screen" >}}
The process is now mostly automated, and you can follow the progress in the Output window.
{{< img src="images/ude/image13.png" alt="Enable Dynamic 365 app on the next screen" >}}

## Further reading
- Microsoft documentation : <a href="https://learn.microsoft.com/en-us/power-platform/developer/unified-experience/finance-operations-install-config-tools" target="_blank">Install and configure development tools</a> 
- Next post :  "How to deploy your code to the UDE environment"
