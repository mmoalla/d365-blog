---
title: "Electronic Reporting in D365 F&O - Part 2: model mapping"
date: 2026-08-20
draft: false
categories: ["Functional configuration"]
tags: ["Electronic Reporting", "ER", "Configuration"]
summary: "The data model we built in Part 1 is just an abstract shape with no real data behind it. Model mapping is what connects it to an actual D365 table - here's how to wire SalesOrder Model to SalesTable, field by field."
---
 
## Context
 
In [Part 1 of this series](/posts/electronic-reporting-part1/), we built `Sales Order Model`: a data model with a root node `SalesOrderModel` and two fields, `SalesId` and `CustAccount`. At that point, the model is purely structural — it describes *what* the document should contain, but it has no idea *where* that data actually comes from in D365.

In this post we will configure the model datasource mapping and map the model previously created to corresponding data.

## Prerequisites
 
The `Sales Order Model` configuration from Part 1, with status **Completed**
- The same **Electronic reporting developer** security role used in Part 1
- Access to **Organization administration > Workspaces > Electronic reporting**.

## Step 1: Create a model mapping designer configuration
1. In **Electronic reporting**, select `Sales Order Model` configuration.
2. Click **Designer** button
3. In SalesOrderModel designer screen, click **Map model to datasource**.
4. In *Model to datasource mapping* screen, click **New** button to add a mapping.
{{< img src="images/ER/part2/image1.png" >}}
5. Choose the **Definition** `SalesOrderModel` created in part 1 and enter a name.
6. Click **Save**.
7. In the same screen *Model to datasource mapping*, Click **Designer** button.
In *Model mapping designer* screen is divided on 3 panels: 
    - The left panel show different **Datasource types**: Table records, Enumeration, Class, Calculated field, User input parameter and so on.
    - The middle panel show **Datasources** used to show data in the report.
    - The right panel contain the **Data model** that we configure on the part 1.
8. Select **Table records** in left panel and click **Add root**
{{< img src="images/ER/part2/image2.png" >}}
9. Select **SalesTable** table and enter a name *SalesTable*.
10. Click **OK**.
{{< img src="images/ER/part2/image3.png" >}}

## Step 2: Map data sources to data model
1. Select **SalesTable** Table records in data sources panel.
2. Select **Sales Order** Record List in data model panel. 
3. Click **Bind**
{{< img src="images/ER/part2/image4.png" >}}
*The Sales Order record list will now in bold. This means that it is binded to salesTable Records*.
4. Expend **SalesTable** Table records, find *Customer account* table field and select it.
5. Expend **Sales Order** Record List and select *CustAccount* node.
6. Click **Bind**.
{{< img src="images/ER/part2/image5.png" >}}
7. Repeat the operation for SalesId node:
    - Sales Order with SalesId.
8. Click **Save**.
9. Return to **Electronic reporting**, select `Sales Order Model` configuration and then select **Change status** to move the configuration from **Draft** to **Completed**.

## Further reading
- [Overview of Electronic Reporting (ER) — Microsoft Learn](https://learn.microsoft.com/dynamics365/fin-ops-core/dev-itpro/analytics/general-electronic-reporting).
- [Electronic reporting data sources — Microsoft Learn](https://learn.microsoft.com/en-us/dynamics365/fin-ops-core/dev-itpro/analytics/er-data-sources).
- **Part 2 of this series**: model mapping, to link `Sales Order Model` to the real data in `SalesTable`.