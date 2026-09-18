---
title: "Electronic Reporting in D365 F&O - Part 3: Format designer and generating the output"
date: 2026-08-23
draft: false
categories: ["Functional configuration"]
tags: ["Electronic Reporting", "ER", "Configuration"]
summary: "The model and the mapping are ready - now it's time to actually produce a file. Here's how to design an Excel format for Sales Order Model, bind it to the model, and generate a real export."
series: "Electronic Reporting"
lightbox:
  enabled: true
justified_gallery:
  enabled: true
---
 
# Context
In [Part 1]({{% ref path="posts/ER/electronic-reporting-part1" lang="en" %}}), we built `Sales Order Model`, an abstract structure with `SalesId` and `CustAccount`. In [Part 2]({{% ref path="posts/ER/electronic-reporting-part2" lang="en" %}}), we connected that structure to real data by mapping it to `SalesTable`.
 
Neither of those two pieces produces an actual file on their own. That's the role of the **format**: it describes the physical shape of the output (an XML document, a CSV file, an Excel workbook...), and it's bound field by field to the data model — not directly to the table. This last layer is what finally turns "abstract data" into "a file you can open."
 
In this part, we'll build a simple Excel format for `Sales Order Model` and generate a real export.

# Prerequisites
- The `Sales Order Model` configuration from Part 1, status **Completed**.
- The `Sales Order Model Mapping` configuration from Part 2, status **Completed**.
- Access to **Organization administration > Workspaces > Electronic reporting**.

# Steps
## Create file template
1. Open new Excel file and create a table that contains two columns *Order number* and *Customer account*
![](images/ER/part3/image1.png)

2. Define *name* to table:
    - In **Formulas** Excel tabs, click **Name manager** in *Defined name section*.
    - Select Table line and click **Edit** button.
    ![](images/ER/part3/image2.png)
    - Put a consistent name *SalesOrders* and click **OK** button
    ![](images/ER/part3/image3.png)
3. Define name to *Order number* table column and all Sheet Columns if necessary.
![](images/ER/part3/image4.png)
4. Repeat operation for *Customer account* table column.
5. Save excel template.

## Create a format configuration
1. In **Electronic reporting**, select `Sales Order Model` configuration.
2. Select **Create configuration**.
3. Enter a name.
4. Select **Excel** format type.
5. Select the last **Data model version** and select the **the Data model definition**.
![](images/ER/part3/image5.png)
6. Click **Create configuration** button.
![](images/ER/part3/image6.png)

## Attach the template to format
Attach Excel template created in step 1 to *Sales order (Excel)* format.
1. Select *Sales order (Excel)* format and click **attachements** button.
![](images/ER/part3/image7.png)
2. In **attachements** screen click **new** button and click **file**.
![](images/ER/part3/image8.png)
3. Upload the template file.

## Create format range and cells
1. Open **Electronic reporting** screen, select *Sales order (Excel)* format.
2. Click **Designer** button.
3. In *Format designer* screen, open **Add root** dropdown and select **File** in *Excel* category.
![](images/ER/part3/image9.png)
4. Enter a **name**, select Attached template file and click **OK**
![](images/ER/part3/image10.png)
5. Select root node and open **add** dropdown.
6. Select Range (it correpond to template table lines).
![](images/ER/part3/image11.png)
7. Enter a **name** and in **Excel range** filed enter the same name as defined in the template. 
8. Select **Vertical** in replication direction field to show all sales order lines otherwise it show the first line.
![](images/ER/part3/image12.png)
![](images/ER/part3/image13.png)
9. Select the range node, open **add** dropdown and select cell.
![](images/ER/part3/image14.png)
10. Enter a **name** for the first excel cell.
11. In **Excel range** filed enter the same name as defined in the template and select data type (String in our example).
![](images/ER/part3/image15.png)
11. Repeat operation for customer account.

## Map format with model
1. In *Format designer* screen, Click **Mapping** tab in the right.
![](images/ER/part3/image16.png)
2. Select *SalesOrders* format node and *SalesOrder* model node and click **Bind** button.
![](images/ER/part3/image17.png)
3. Repeat operation for sub-nodes.
4. Click **Save**.
5. Return to **Electronic reporting**, select *Sales order (Excel)* format and then select **Change status** to move the configuration from **Draft** to **Completed**.

## Validate and execute the report.
1. Select *Sales order (Excel)* format and click **Validate** button.
2. Click **Run** button and click **ok**
3. Report show all sales order lines in Excel file.
![](images/ER/part3/image18.png)

This is a simple example of how to create an electronic reporting we can also add user parameters, show query panel, add a multiselect lookup, execute report in cross-company and also we can create a menu to execute report outside Electronic Reporting workspace.

# Further reading
- [Overview of Electronic Reporting (ER) — Microsoft Learn](https://learn.microsoft.com/dynamics365/fin-ops-core/dev-itpro/analytics/general-electronic-reporting)
- [Create Electronic Reporting configurations — Microsoft Learn](https://learn.microsoft.com/dynamics365/fin-ops-core/dev-itpro/analytics/electronic-reporting-configuration)