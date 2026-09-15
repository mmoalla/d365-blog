---
title: "Power Automate pagination: retrieving more than 5,000 rows"
date: 2026-09-15
draft: false
categories: ["Power Platform"]
tags: ["Power Automate", "Dataverse", "FetchXML", "Pagination"]
summary: "The Dataverse connector's List rows action is limited to 5,000 rows per call. Here is how to build a robust pagination loop with FetchXML, the paging cookie, and the More Records flag to retrieve a complete dataset, regardless of its size."
---
 
## Context
 
Dataverse imposes a technical limit: **the `List rows` query can never return more than 5,000 rows**, regardless of how you build your query (Filter rows or FetchXML). This is not noticeable with a smaller dataset. For a synchronization, an export, or a bulk operation on a table that exceeds this threshold, the flow silently stops at 5,000 rows without any error.
 
The solution is a loop that requests the next page as long as there are more records, relying on two pieces of information that Dataverse returns with each response:
 
- **`@Microsoft.Dynamics.CRM.morerecords`** — a Boolean indicating whether there are more rows to retrieve.
- **`@Microsoft.Dynamics.CRM.fetchxmlpagingcookie`** — a pagination cursor, more efficient than a simple page number, which must be injected into the next query.

## Prerequisites
 
- A Power Automate flow with a **List rows** action (Dataverse connector) configured in **Fetch XML Query** or **Filter rows** mode

## Step 1: initialize the pagination variables
 
Before the loop, add four variables:
 
| Name | Type | Initial value |
|---|---|---|
| `PageNumber` | Integer | `1` |
| `PagingCookie` | String | *(empty)* |
| `MoreRecords` | Boolean | `true` |
| `TotalRecordsCount` | Integer | `0` |

{{< img src="images/PA/image1.png">}}

## Step 2: Build the loop structure
 
Add a **Do until** action with this condition: `MoreRecords` is equal to `false`.
{{< img src="images/PA/image2.png">}}

**Important point before going any further:** the default number of iterations for a `Do until` is limited (60 by default). If your table may contain more than 300,000 rows, open the action settings (the gear icon) and increase the iteration limit. Otherwise, the loop will stop prematurely without retrieving everything, again without an explicit error.
 
## Step 3: The dynamic FetchXML query
 
Inside the loop, add your **List rows** action with a FetchXML query that dynamically includes the `page` and `paging-cookie` attributes, and sets `count` to 5,000 (the Dataverse limit):
 
```xml
<fetch count="5000" page="@{variables('PageNumber')}" @{if(equals(variables('PageNumber'), 1), '', if(equals(variables('PagingCookie'), ''), '', concat('paging-cookie=''', encodeXmlValue(variables('PagingCookie')), '''')))}>
  <entity name="kpmg_movies">
    <attribute name="kpmg_categories" />
    <attribute name="kpmg_name" />
  </entity>
</fetch>
```
Notice the syntax: `variables('PagingCookie')` is inserted **directly into the `<fetch>` tag**. If the variable is empty, nothing is injected. On subsequent pages, it contains the literal `paging-cookie="..."` attribute.
 
## Step 4: Add a condition to check whether there is data

Still inside the loop, after the **List rows** action:
- Add a condition that checks whether the **List rows** action contains data with this expression:
If **`length(outputs('List_movies')?['body/value'])`** is greater than 0.
{{< img src="images/PA/image3.png">}}

## Step 5: update the control variables
**If the condition is True:**

- Update the **`PagingCookie`** variable.
This is the most delicate step in the entire process: the raw value returned by Dataverse in `@Microsoft.Dynamics.CRM.fetchxmlpagingcookie` cannot be reused as-is. It is encoded and contains characters that must be decoded before being injected into the XML of the next query:
1. Check that the **`@Microsoft.Dynamics.CRM.fetchxmlpagingcookie`** property is not null.
2. Extract only the useful part of the raw string.
3. Decode the special characters (`<`, `>`, `"`) that were escaped in HTML format:
**`decodeUriComponent(decodeUriComponent(split(split(outputs('List_movies')?['body']?['@Microsoft.Dynamics.CRM.fetchxmlpagingcookie'], 'pagingcookie="')[1], '" istracking')[0]))`**

- Increment the **`PageNumber`** variable by 1.
- Set the **`MoreRecords`** variable to *true*.
- Increment the **`RecordsCount`** variable by the length of the results array (`length(outputs('List_rows')?['body/value'])`).

**If the condition is FALSE:**
- Set the **`MoreRecords`** variable to *false*, which stops the `Do until` loop.

{{< img src="images/PA/image4.png">}}

## Step 6: process the results of each page
 
Do not wait until the loop ends to process the data. Add your processing step (updating another table, sending data to an API, writing to a file, and so on) **inside the loop**, immediately after **List rows**, one page at a time.
 
## Common pitfalls
- **Leaving the native "Pagination" setting enabled in the `List rows` settings** while manually handling pagination through `page`/`paging-cookie`: the two mechanisms conflict. Disable the native pagination option when managing everything yourself through FetchXML.
- **Using an `order` clause on an attribute of a related entity (`link-entity`)**: some queries sorted this way do not support the paging cookie, and Dataverse silently falls back to simple page-number pagination. This is less efficient and worth monitoring if your results seem inconsistent between two pages.
- **Forgetting to increase the `Do until` iteration limit** for a very large volume: the default value is designed for simple scenarios, not exports involving several hundred thousand rows.
- **Treating a missing `MoreRecords` value as `true` by default**: if the expression that reads this field does not handle the case where it is empty or missing, the loop may never end.

## Further reading
- [Paginate through large result sets with FetchXML — Microsoft Learn](https://learn.microsoft.com/power-apps/developer/data-platform/fetchxml/page-large-result-sets)
- [Page large result sets (Dataverse) in Power Automate — Power Platform Community](https://community.powerplatform.com/blogs/post/?postid=0864e145-da42-4922-b667-ff10dd160089)