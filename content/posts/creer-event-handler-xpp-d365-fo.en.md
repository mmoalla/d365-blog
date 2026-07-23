---
title: "Create an Event Handler in X++ for Dynamics 365 F&O"
date: 2026-07-22
draft: false
categories: ["X++ & Dev"]
tags: ["X++", "Event Handler", "Extensions", "Chain of Command"]
summary: "How to properly intercept a standard event in D365 F&O without changing standard code, with a concrete example on the SalesTable."
---

## Context
In Dynamics 365 Finance & Operations, directly modifying Microsoft standard code is prohibited (the "over-layering" model disappeared after AX 2012). All customizations must be done by **extensions**.

Two main mechanisms exist to intervene on existing code:

- **Chain of Command (CoC)** : To extend the behavior of an existing method (before/after/around its execution).
- **Event Handlers** : To react to specific events (`Created`, `Updated`, `Deleted`, `ValidateWrite`...) without changing the original logic.

This tutorial focuses on **Event Handlers**, often the simplest and safest choice for needs such as: triggering a notification, writing a log, enforcing an additional business rule, or synchronizing data to an external system.

## Prerequisites
- Dynamics 365 F&O version 10.0.x (Tier 1 dev box, cloud-hosted environment, or UDE).
- Visual Studio.
- Write access to the model and access to the Application Explorer.

## Steps
### 1. Create the project and the extension class
In Visual Studio, create a new D365 F&O project, then add an event class:

```xpp
public class CustTableHandler
{
}
```

### 2. Identify the event to intercept
Open the `CustTable` table in the Application Explorer and locate the `confirmAndSaveCustGroupChange()` method. Here we want to execute an action **after** a sales order is created.

{{< img src="images/eventHandler/SalesTable.png" alt="Sales table object" >}}
{{< img src="images/eventHandler/SalesCopyEvent.png" alt="Sales table object" >}}

Two event options exist for most standard methods:
- `[PostHandlerFor]` — runs after the original method.
- `[PreHandlerFor]` — runs before the original method.

### 3. Write the Post Event Handler
```xpp
class CustTableHandler
{
    [PreHandlerFor(tableStr(CustTable), tableStaticMethodStr(CustTable, confirmAndSaveCustGroupChange))]
    public static void CustTable_Post_confirmAndSaveCustGroupChange(XppPrePostArgs args)
    {
        //Object instance of the CustTable table
        CustTable custTable = args.getThis();
        //parameter value in confirmAndSaveCustGroupChange method
        boolean isCustGroupSetOnce = args.getArg('isCustGroupSetOnce');
        //return value of the confirmAndSaveCustGroupChange method
        boolean valueModifiedReturn = args.getReturnValue();
    }
}
```

**Key points:**
- `args.getThis()`  retrieves the instance of the object (here the `SalesTable` record that was just inserted).
- `args.getArg('isCustGroupSetOnce')` retreives the method parameters.
- `args.getReturnValue()` retreives the return method value
- The handler must be `static` and accept a `XppPrePostArgs` parameter.
- The method name has no functional importance, but the `Table_Post_methode` convention improves readability

### 4. Add validation with a Pre Event Handler
To block an operation before it happens (e.g., prevent creating an order for an inactive customer):

```xpp
[PreHandlerFor(tableStr(SalesTable), tableMethodStr(SalesTable, insert))]
public static void SalesTable_Pre_insert(XppPrePostArgs args)
{
    SalesTable salesTable = args.getThis();
    CustTable custTable = CustTable::find(salesTable.CustAccount);

    if (custTable.Blocked != CustVendorBlocked::No)
    {
        throw error(strFmt("Customer account %1 is blocked.", salesTable.CustAccount));
    }
}
```

### 5. Build and test
- Build the project (Build or Rebuild)
- Test by creating a sales order manually

## Common pitfalls
- **Multiple event handlers on the same event** : execution order between extensions from different models is not guaranteed; avoid relying on order.

## Further reading
- Microsoft documentation : <a href="https://learn.microsoft.com/fr-fr/dynamics365/fin-ops-core/dev-itpro/dev-ref/xpp-events" target="_blank">Event Handeler</a> 
- Next post :  "Chain of Command in X++: when to use it instead of an Event Handler"
