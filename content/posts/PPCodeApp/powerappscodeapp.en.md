---
title: "Power Apps Code Apps: Create pro-code React apps into the Power Platform"
date: 2026-08-25
draft: false
categories: ["Power Platform"]
tags: ["Power Apps", "Code Apps", "React", "TypeScript", "Dataverse"]
summary: "Code Apps let you write a full React/TypeScript web application in your own IDE and run it inside Power Platform, with typed access to Dataverse and 1,500+ connectors - no drag-and-drop canvas, no PCF boilerplate. Here's how to scaffold, connect, and run your first one."
---
 
## Context
 
Canvas apps and model-driven apps are built through Power Apps studio. **Code Apps** take a different route entirely: you write a standalone **React** in a real IDE like VS Code, and the Power Platform CLI wires it into the platform - authentication, data connections, and hosting are handled for you, while you keep full control over the UI and logic in actual code.
 
Microsoft announced General Availability for Code Apps in February 2026. It's worth being precise about what they are and aren't:
 
- They are **not** canvas apps, **not** model-driven apps, and **not** PCF controls
- They **are** standalone web applications - built with *Vite, React, and TypeScript* by default - that happen to run inside the Power Apps runtime
- They get typed access to **Dataverse and 1,500+ connectors** directly from *JavaScript/TypeScript*, without writing authentication code yourself.

If you're a developer who's always found canvas apps too limiting but didn't want to give up the governance and connector ecosystem of Power Platform, this is the option built for that gap.
 
## Prerequisites
 
- [Visual Studio Code](https://code.visualstudio.com/download).
- [Node.js](https://nodejs.org/).
- The **Power Platform CLI** (`pac`).
- A Power Platform environment with Code Apps enabled.
- A **Power Apps Premium** license.

## Step 1: Enable Code Apps feature.
1. Open [Power Platform Admin Center](https://admin.powerplatform.microsoft.com/).
2. Select environment in wich you will create your code app.
3. Click **Setting**.
4. In **Product** menu, click **Features** sub menu.
5 Scroll Down and enable **Power Apps Code Apps** feature.
{{< img src="images/PPCodeApp/image1.png" >}}

## Step 2: Scaffold the project from the official template
 
```bash
npx degit github:microsoft/PowerAppsCodeApps/templates/vite my-code-app
cd my-code-app
```
 
This gives you a clean Vite + React + TypeScript project with the Power Apps SDK already wired in - no manual setup of the SDK plumbing required.
{{< img src="images/PPCodeApp/image2.png" >}}
 
## Step 2: Authenticate and select your target environment
 
```bash
pac auth create
pac env list
pac env select --environment <your-environment-id>
```
{{< img src="images/PPCodeApp/image3.png" >}}
 
## Step 3: Install dependencies and initialize the app
 
```bash
npm install
pac code init --displayname "My First Code App"
```
{{< img src="images/PPCodeApp/image4.png" >}}

`pac code init` registers the app in your Power Platform environment and generates a `power.config.json` file - this is what links your local project to the platform for both data access and later deployment.
{{< img src="images/PPCodeApp/image5.png" >}}

## Step 4: Connect a data source

This is where Code Apps really differ from a plain React app: instead of hand-writing API calls, you generate typed access to any Power Platform connector directly from the CLI.
```bash
pac code add-data-source -a "shared_office365users" -c "<connector_id>"
```
This command generate the following files:
{{< img src="images/PPCodeApp/image6.png" >}}
 
## Step 5: Run it locally
 
Because Code Apps are built on Vite, local development works exactly like any modern React project — hot reload included:
 
```bash
npm run dev
```
The app runs against the real connected data source while you develop, so what you see locally matches what runs once deployed.
{{< img src="images/PPCodeApp/image7.png" >}}

## Step 6: Deploy
 
Once you're happy with the app, `pac code push` (or the equivalent deployment command for your CLI version) publishes it to your Power Platform environment, where it becomes available like any other Power Platform app — governed by the same sharing, security, and DLP policies as everything else in your tenant.

**In the next post, we'll dive into some React fundamentals and see how to display the current user's profile in a Power Apps Code App using React.**

## Further reading
- [Power Apps code apps overview — Microsoft Learn](https://learn.microsoft.com/power-apps/developer/code-apps/overview)
- [Quickstart: Create a code app from scratch — Microsoft Learn](https://learn.microsoft.com/power-apps/developer/code-apps/how-to/create-an-app-from-scratch)