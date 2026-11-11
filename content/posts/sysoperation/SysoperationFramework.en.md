---
title: "SysOperation framework in X++: batch jobs, parameters, queries and validation"
date: 2026-11-11
draft: false
categories: ["X++ & Dev"]
tags: ["X++", "SysOperation", "Batch", "Data Contract", "Validation"]
summary: "The SysOperation framework is the standard way to build processes in D365 F&O that can run interactively or in batch. Here's how the contract, service and controller fit together, and how to add parameters, a query, validation and batch execution."
lightbox:
  enabled: true
justified_gallery:
  enabled: true
---
 
# Context
 
Any time a process in D365 F&O needs a dialog for the user, the ability to run in the background, or the ability to be scheduled as a recurring batch job, you want the **SysOperation framework**. It replaces the older RunBase framework, and Microsoft's own guidance for finance and operations apps is to use it whenever a job needs to process or calculate something.
 
Its core idea is a clean separation of responsibilities across a few classes:
 
| Piece | Role |
|---|---|
| **Data contract** | Defines the parameters of the process, and where validation lives. The dialog is generated automatically from it |
| **Service** | Contains the actual business logic — the code that does the work |
| **Controller** | Wires the contract and the service together, and decides how the operation executes (interactive, batch...) |
| **UI builder** *(optional)* | Customizes the generated dialog when the default isn't enough |
 
Because each piece is a normal class, each one stays small, testable, and — importantly for D365 — extensible through Chain of Command and event handlers, which was much harder with RunBase.
 
# Prerequisites
 
- Dynamics 365 F&O development environment with Visual Studio
- A custom model to hold your classes
- Basic familiarity with X++ classes and attributes

# Steps
## The data contract — your parameters
 
The contract is a class marked with `[DataContract]`. Each parameter is a private variable exposed through a **parm method** decorated with `[DataMember]`.
 
```xpp
[DataContract]
class MyCustomerProcessContract implements SysOperationValidatable
{
    CustGroupId custGroupId;
    TransDate   cutoffDate;
    NoYes       includeInactive;
 
    [
        DataMember,
        SysOperationLabel("Customer group"),
        SysOperationDisplayOrder('1')
    ]
    public CustGroupId parmCustGroupId(CustGroupId _custGroupId = custGroupId)
    {
        custGroupId = _custGroupId;
        return custGroupId;
    }
 
    [
        DataMember,
        SysOperationLabel("Cutoff date"),
        SysOperationDisplayOrder('2')
    ]
    public TransDate parmCutoffDate(TransDate _cutoffDate = cutoffDate)
    {
        cutoffDate = _cutoffDate;
        return cutoffDate;
    }
 
    [
        DataMember,
        SysOperationLabel("Include inactive customers"),
        SysOperationDisplayOrder('3')
    ]
    public NoYes parmIncludeInactive(NoYes _includeInactive = includeInactive)
    {
        includeInactive = _includeInactive;
        return includeInactive;
    }
}
```
 
**What each attribute does:**
- `[DataContract]` / `[DataMember]` — tell the framework this class holds parameters, and which parm methods are parameters. Only parm methods with `[DataMember]` show up in the dialog and get saved with the batch job
- `SysOperationLabel` — the caption shown in the dialog (use a label reference in real code rather than a hard-coded string)
- `SysOperationDisplayOrder` — controls the order of fields in the dialog
A few more attributes are worth knowing: `SysOperationGroup` and `SysOperationGroupMember` put parameters into named groups in the dialog, and `SysOperationControlVisibility(false)` hides a parameter from the UI while still keeping it in the contract — handy for values passed in programmatically.
 
**Default values:** to give parameters initial values, implement `SysOperationInitializable` and fill them in its `initialize()` method. Adding `[SysOperationAlwaysInitialize]` to the contract makes the framework call it every time, rather than only when there are no saved last values.
 
## The service — your business logic
 
```xpp
class MyCustomerProcessService extends SysOperationServiceBase
{
    [SysEntryPointAttribute(false)]
    public void process(MyCustomerProcessContract _contract)
    {
        CustGroupId custGroupId = _contract.parmCustGroupId();
        TransDate   cutoffDate  = _contract.parmCutoffDate();
 
        CustTable custTable;
 
        while select custTable
            where custTable.CustGroup == custGroupId
        {
            // ... the real work happens here
            info(strFmt("Processed customer %1 up to %2", custTable.AccountNum, cutoffDate));
        }
    }
}
```
 
The service operation receives the contract as its parameter — that's how the values the user typed into the dialog (or that were saved with the batch job) reach your code. Keep this class focused on *doing the work*: no dialog logic, no execution-mode logic. That's exactly what the other pieces are for.
 
## The controller — tying it together
 
```xpp
class MyCustomerProcessController extends SysOperationServiceController
{
    protected void new()
    {
        super(
            classStr(MyCustomerProcessService),
            methodStr(MyCustomerProcessService, process),
            SysOperationExecutionMode::Synchronous);
    }
 
    public ClassDescription defaultCaption()
    {
        return "Process customers";
    }
 
    public static MyCustomerProcessController construct(
        SysOperationExecutionMode _executionMode = SysOperationExecutionMode::Synchronous)
    {
        MyCustomerProcessController controller = new MyCustomerProcessController();
        controller.parmExecutionMode(_executionMode);
        return controller;
    }
 
    public static void main(Args _args)
    {
        MyCustomerProcessController controller = MyCustomerProcessController::construct();
        controller.parmArgs(_args);
        controller.startOperation();
    }
}
```
 
The controller's `new()` tells the framework **which service class and which method** to run. `main()` is the entry point a menu item calls: it builds the controller, hands it the arguments, and `startOperation()` shows the dialog and runs the operation.
 
To expose the process to users, create an **Action menu item** whose object type is *Class* and whose object is the controller (`MyCustomerProcessController`).
 
## Running in batch
 
Because the framework generates the dialog, batch support comes almost for free: the generated dialog has a **Batch** tab, and the standard **Run in the background** option, so users can run the same process interactively or schedule it as a recurring job without any extra code from you.
 
The behavior is driven by the **execution mode**:
 
| Mode | Behavior |
|---|---|
| `Synchronous` | Runs immediately, the user waits for it to finish |
| `Asynchronous` | Runs immediately but doesn't block the user's session |
| `ReliableAsynchronous` | Runs asynchronously through the batch infrastructure, with reliability guarantees |
| `ScheduledBatch` | Always goes through the batch framework |
 
You can force a process to always run in batch by constructing the controller with `ScheduledBatch`, or let the user decide through the dialog.
 
**Creating batch tasks programmatically** — useful when one batch job must launch several steps:
 
```xpp
MyCustomerProcessController controller = MyCustomerProcessController::construct();
controller.parmShowDialog(false);
controller.parmLoadFromSysLastValue(false);
 
batchHeader.addTask(controller);
```
 
`parmShowDialog(false)` skips the dialog (the parameters are set in code instead), and `parmLoadFromSysLastValue(false)` stops the controller from reloading whatever the user last typed. If the batch tasks form needs to let users edit parameters per task, decorate the contract with `[SysOperationJournaledParametersAttribute(true)]`.
 
## Adding a query parameter
 
Often a batch job should let the user filter *which records* to process, using the same advanced filter dialog found elsewhere in the application. A query is added to the contract as a packed string:
 
```xpp
[DataContract]
class MyCustomerProcessContract implements SysOperationValidatable
{
    str packedQuery;
 
    [
        DataMember,
        AifQueryType('_packedQuery', queryStr(CustTableQuery))
    ]
    public str parmQuery(str _packedQuery = packedQuery)
    {
        packedQuery = _packedQuery;
        return packedQuery;
    }
 
    public Query getQuery()
    {
        return new Query(SysOperationHelper::base64Decode(packedQuery));
    }
 
    public void setQuery(Query _query)
    {
        packedQuery = SysOperationHelper::base64Encode(_query.pack());
    }
}
```
 
`AifQueryType` points the parameter at an existing AOT query (replace `CustTableQuery` with whichever query fits your table) — it defines which tables and fields the user can filter on in the dialog. In the service, read the user's filter back and run it:
 
```xpp
public void process(MyCustomerProcessContract _contract)
{
    Query    query    = _contract.getQuery();
    QueryRun queryRun = new QueryRun(query);
    CustTable custTable;
 
    while (queryRun.next())
    {
        custTable = queryRun.get(tableNum(CustTable));
        // ... process the filtered record
    }
}
```
 
Because the query is stored **packed** inside the contract, it travels with the batch job exactly like any other parameter: whatever filter the user defined when scheduling the job is the filter used every time it runs.
 
## Validation
 
Implementing `SysOperationValidatable` on the contract (as we did above) lets you add a `validate()` method. The framework calls it when the user confirms the dialog, **before** the operation runs or the batch job is created — so invalid input never reaches your service:
 
```xpp
public boolean validate()
{
    boolean ret = true;
 
    if (!custGroupId)
    {
        ret = checkFailed("Customer group is mandatory.");
    }
 
    if (cutoffDate > DateTimeUtil::getToday(DateTimeUtil::getUserPreferredTimeZone()))
    {
        ret = checkFailed("The cutoff date cannot be in the future.");
    }
 
    return ret;
}
```
 
`checkFailed` posts the message to the Infolog **and** returns `false` — the pattern used throughout the standard application for validation. Notice that both checks run even if the first one fails, so the user sees *all* the problems at once rather than fixing them one at a time.
 
Put cross-field rules here, in the contract. Keep validation out of the service: by the time the service runs — possibly hours later in a batch — it's too late to ask the user to correct anything.
 
## Customizing the generated dialog
 
For most processes the automatic dialog is enough, but you can hook in your own **UI builder** class when you need more control — for example, enabling or disabling a field depending on another field's value. You register it on the contract with `SysOperationContractProcessing(classStr(MyUIBuilder))`, and the builder can add, modify or react to dialog controls. Reach for it only when attributes like `SysOperationLabel`, `SysOperationGroup` and `SysOperationControlVisibility` aren't enough.
 
## Common pitfalls
 
- **Forgetting `[DataMember]` on a parm method**: the parameter silently won't appear in the dialog, and won't be saved with the batch job.
- **Putting business logic in the controller**: the controller should only wire things together and decide how to execute; logic placed there is harder to test and to extend than logic placed in the service.
- **Validating in the service instead of the contract**: in a batch scenario, the service may run long after the user has left — validation belongs in the contract, at submission time.
- **Heavy work without batch support in mind**: a process that touches a lot of data should be designed to be batch-friendly from the start (idempotent, restartable, processing in sensible chunks) rather than retrofitted later.
- **Hard-coded captions**: labels in `SysOperationLabel` should be label references, not literal strings, so the dialog is translated like the rest of the application.
- **Assuming the batch job re-asks for parameters**: the values saved with a batch job are the values it will use on every run — if the user needs different values, they must edit the job, not expect a new dialog.
## Further reading
 
- [Finance and operations apps frameworks — Microsoft Learn](https://learn.microsoft.com/training/modules/design-deployment-finance-operations/8-frameworks)