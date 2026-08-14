---
title: "Chain of Command on Classes, Tables, Forms and Data Entities: the complete guide"
date: 2026-07-13
draft: false
categories: ["X++ & Dev"]
tags: ["X++", "Chain of Command", "Extensions"]
summary: "Chain of Command isn't just for classes and tables: forms, data sources, fields, controls, and data entities each have their own extension syntax. Here's a complete tour, with one example for each case."
---
 
## Context
 
**Chain of Command (CoC)**, alongside Event Handlers, is one of the two main extensibility mechanisms in Dynamics 365 F&O. Unlike Event Handlers, CoC lets you **change the behavior** of an existing method: alter its return value, transform its parameters, or even block its execution entirely.
 
The syntax relies on three elements:
- A `final` class, marked with the `[ExtensionOf(...)]` attribute that indicates which object it extends.
- A method that reproduces **exactly** the signature of the original method (same name, same parameters, same return type).
- The `next` keyword, which calls the original method (or the previous extension in the chain) — the equivalent of `base()` in C#

```xpp
[ExtensionOf(classStr(SalesFormLetter))]
final class MonEntreprise_SalesFormLetter_Extension
{
    public void update(SalesFormLetter_Invoice _formLetter)
    {
        // Logic before the original execution
        next update(_formLetter);
        // Logic after the original execution
    }
}
```
 
What doesn't get covered often is that **every type of X++ object has its own targeting function** inside the `[ExtensionOf(...)]` attribute. Using the wrong function for the wrong object is the most common mistake when discovering CoC on forms — compilation often fails without a very clear message about the actual cause.
 
Here's the function to use for each of the seven most common extension points:
 
| Object to extend | Targeting function |
|---|---|
| Class | `classStr(ClassName)` |
| Table | `tableStr(TableName)` |
| Form (methods on the form itself) | `formStr(FormName)` |
| Form data source | `formDataSourceStr(FormName, DataSourceName)` |
| Data field on a form | `formDataFieldStr(FormName, DataSourceName, FieldName)` |
| Form control | `formControlStr(FormName, ControlName)` |
| Data Entity | `tableStr(DataEntityName)` (a data entity is technically a table) |
 
## Prerequisites
 
- Dynamics 365 F&O version 10.0.x.
- Visual Studio with the D365 F&O development tools.
- An existing custom model.
- Being familiar with the basics of Chain of Command (recommended).
## 1. Extending a class
 
The principle is the one we just saw in the introduction. Another common example: intercepting an amount calculation in a standard business class.
 
```xpp
[ExtensionOf(classStr(CustPaymSchedRule))]
final class PrefixCompanyCustPaymSchedRule_Extension
{
    public AmountMST calcPaymAmount(AmountMST _amount, int _numberOfPayments)
    {
        AmountMST amount = next calcPaymAmount(_amount, _numberOfPayments);
 
        if (this.RoundUp)
        {
            amount = PrefixCompanyRoundingHelper::roundUp(amount);
        }
 
        return amount;
    }
}
```
 
## 2. Extending a table
 
```xpp
[ExtensionOf(tableStr(CustTable))]
final class CustTable_Prefix_T_Extension
{
    void initValue()
    {
        next initValue();

        //set your new custom code
    }
}
```
 
## 3. Extending a form
 
Here we target the form itself — useful for hooking into its global methods like `init()`:
 
```xpp
[ExtensionOf(formStr(CustTable))]
final class CustTable_Prefix_F_Extension
{
    public void init()
    {
        next init();

        //customize code goes here
    }
}
```
 
## 4. Extending a form data source
 
Each `FormDataSource` (the link between a form and a table) can be extended independently — useful for hooking into record loading, validation, or activation:
 
```xpp
[ExtensionOf(formDataSourceStr(CustTable, CustTable))]
final class CustTable_Prefix_DS_Extension
{
    public int active()
    {
        int ret;
        ret = next active();

        CustTable custTableLocal = this.cursor();

        //Based on the currently selected record,
        //enable/disable or show/hide your button or control here.

        return ret;
    }
}
```
 
**Important note:** the `FormDataSourceName` parameter in `formDataSourceStr(FormName, FormDataSourceName)` refers to the **name of the data source in the form's tree view**, not necessarily the name of the underlying table — the two are often identical, but not always (a data source can be renamed in the designer).
 
## 5. Extending a data field on a form
 
```xpp
[ExtensionOf(formDataFieldStr(CustTable, CustTable, CustGroup)]
final class CustTable_Prefix_DF_Extension
{
    public void modified()
    {
        FormDataObject formDataObject = any2Object(this) as FormDataObject;
        FormDataSource formDataSource = formDataObject.datasource();
        CustTable custTable;

        next modified();

        custTable = formDataSource.cursor();
        custTable.CustGroup = CustGroup::find(custTable.CustGroup).CustGroup;

    }
}
```
 
## 6. Extending a form control
 
```xpp
[ExtensionOf(formControlStr(CustTable, ButtonDelete))]
final class CustTable_Prefix_FC_Extension
{
    public void clicked()
    {
        FormControl formButtonControl = any2Object(this) as FormControl;
        FormDataSource formDatasource = formButtonControl.formRun().dataSource(tableStr(CustTable));
        CustTable custTable = formDatasource.cursor();

        next clicked();

        info(strFmt("Cust Account %1", custTable.AccountNum));
    }
}
```
 
## 7. Extending a Data Entity
 
```xpp
[ExtensionOf(tableStr(CustomersV3Entity))]
final class CustomersV3Entity_Prefix_Entity_Extension
{
    public boolean validateWrite()
    {
        boolean ret = next validateWrite();

        //Custom code
        if(CustTable.CustGroup != 'Group1')
        {
            ret = checkFailed("message");
        }

        return ret;
    }
}
```
 
This is particularly useful for adding validation or transformation logic during data imports through the Data Management Framework, without touching the entity's standard definition.
 
## Common pitfalls
 
- **Forgetting that the data source name can differ from the table name**: always check the exact name shown in the form designer's tree view, not just the table name.
- **Not handling the case where `next` is never called on a control**: as in the "Delete" button example, this is intentional and legitimate, but make sure it's actually intended and not an oversight — otherwise clicking a standard button can appear to "do nothing" without a clear error message for the user.
- **Extending a Data Entity thinking it behaves differently from a table**: it doesn't — it follows exactly the same CoC rules as a standard table.
## Further reading
 
- [Microsoft Learn documentation: method wrapping and Chain of Command](https://learn.microsoft.com/dynamics365/fin-ops-core/dev-itpro/extensibility/method-wrapping-coc)