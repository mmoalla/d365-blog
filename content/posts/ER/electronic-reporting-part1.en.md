---
title: "Electronic Reporting in D365 F&O - Part 1: foundations and the data model"
date: 2026-08-17
draft: false
categories: ["Functional configuration"]
tags: ["Electronic Reporting", "ER", "Configuration"]
summary: "Electronic Reporting (ER) lets you generate or import regulatory documents without writing code. First installment of a series: key concepts, setting up configuration providers, and building a data model step by step."
series: "Electronic Reporting"
---
 
## Context
**Electronic Reporting (ER)** is the D365 F&O tool used to generate electronic documents (invoices, bank statements, tax document, regulatory exports...) or import them, **without writing a single line of X++ code** in most cases. It's the same tool Microsoft uses internally to deliver most payment formats and local compliance features (SEPA, banking formats, country-specific tax document...).
 
This topic is broad enough to deserve several articles. Here's the plan for the series:
- **Part 1**: key concepts, setting up configuration providers, building a data model.
- **Part 2**: model mapping (linking the data model to real D365 tables).
- **Part 3**: the format designer, format mapping, and generating the final document.

## The key concepts to understand before starting
ER relies on a chain of components, each with a precise role:
 
| Component | Role |
|---|---|
| **Configuration provider** | Identifies who owns/maintains an ER configuration (Microsoft, a country, or your own company) |
| **Data model** | An abstract, business-level representation of what the document should contain — independent of D365's table structure |
| **Model mapping** | Links the abstract data model to real D365 tables, fields, or queries |
| **Format** | Describes the structure of the output file (supported formats: TEXT, XML, JSON, PDF, Microsoft Word, Microsoft Excel, and OPENXML) |
| **Format mapping** | Links each element of the format to the data model |
 
**Why this layered separation?** Because the same data model (e.g. "Sales Order") can be reused by several different formats (an XML export for one country, a CSV for another), and a model mapping can be maintained independently of D365 version upgrades — this is what lets Microsoft ship configuration updates without breaking your customizations.
 
## Prerequisites 
- Dynamics 365 F&O version 10.0.x.
- The **Electronic reporting developer** security role (or an admin role that includes these rights).
- Access to **Organization administration > Workspaces > Electronic reporting**.

## Step 1: create and activate a configuration provider
Before creating anything else, you need an active configuration provider — it "owns" your future configurations.

1. Go to **Organization administration > Workspaces > Electronic reporting**.
2. Select **Configuration providers** from the related links.
{{< img src="images/ER/part1/image1.png" alt="config providers" >}}
3. Select **New**.
4. Enter a **Name** (e.g. your company's name) and your company's **URL**.
{{< img src="images/ER/part1/image2.png" alt="config providers parameters" >}}
5. On the workspace, you'll see two configuration tiles.
6. Select the three dots on your company's tile and select Set active.
{{< img src="images/ER/part1/image3.png" alt="config providers set active" >}}.

## Step 2: create a data model
Let's use a concrete, deliberately simple example for this first part: a data model to export sales orders.
 
1. In **Electronic reporting** workspace, select **Reporting configurations** tile.
2. Select **Create configuration**.
3. Choose **Root** (to create a new model from scratch, without starting from an existing one).
4. Fill in:
   - **Name**: `Sales Order Model`.
   - **Description**: `Data model for exporting the sales orders`.
5. The **Configuration provider** field is automatically filled with the one you activated in step 1.
6. Select **Create configuration**.
{{< img src="images/ER/part1/image4.png" alt="config providers set active" >}}.

### Designing the model's structure 
1. Select the freshly created `Sales Order Model` configuration.
2. Click **Designer** button.
{{< img src="images/ER/part1/image5.png" alt="select model designer" >}}
3. In the designer, select **New** to add the root node:
   - **Name**: `SalesOrderModel`.
{{< img src="images/ER/part1/image6.png" alt="add root" >}}
4. Select this root node, then select **New** again to add a first child node `SalesOrder` of type `Record List`.
This node show list of sales orders.
{{< img src="images/ER/part1/image9.png" alt="add root" >}}
5. Select `SalesOrder` node , then select **New** again to add a first child node field:
   - **Name**: `SalesId`.
   - **Type**: String.
{{< img src="images/ER/part1/image7.png" alt="add root" >}}
6. Repeat the operation to add:
   - **Name**: `CustAccount`.
   - **Type**: String.
7. Select **Save**.
8. Close the designer, then select **Change status** to move the configuration from **Draft** to **Completed**.
{{< img src="images/ER/part1/image8.png" alt="chage status" >}}
**Why go through "Change status"?** A configuration in "Draft" status is still being edited. It needs to move to at least "Completed" for it to become selectable as a source in the next steps (mapping and format).
 
## Common pitfalls
- **Forgetting to activate the configuration provider** before creating a configuration — this is the number one mistake for anyone discovering ER for the first time.
- **Confusing the data model with the format**: the model describes the content in abstract business terms (e.g. "Sales Order Model"), while the format describes the physical structure of the file (e.g. at which position, with which separator). The two are intentionally kept separate.
- **Leaving a configuration in "Draft" status**: it will stay invisible in the selection lists of other components until it's moved to a more advanced status via "Change status".
- **Creating a new model for every need** instead of checking whether Microsoft already provides something close to your scenario (payments, invoices...) — in a real project, it's usually better to look for an existing configuration to extend rather than starting from scratch.

## Further reading
- [Overview of Electronic Reporting (ER) — Microsoft Learn](https://learn.microsoft.com/dynamics365/fin-ops-core/dev-itpro/analytics/general-electronic-reporting).
- [Create Electronic Reporting configurations — Microsoft Learn](https://learn.microsoft.com/dynamics365/fin-ops-core/dev-itpro/analytics/electronic-reporting-configuration).
- **Part 2 of this series**: model mapping, to link `Sales Order Model` to the real data in `SalesTable`.