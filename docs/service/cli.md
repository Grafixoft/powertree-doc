---
id: cli
title: Command-Line Interface
---

PowerTree provides a collection of PowerShell cmdlets used to manage workflow execution.

The first step to get started with them is to follow the [ExecutionEnvironment](configuration) page. 
Then open a PowerShell window and import the `Workflow.Cmdlets` module - `Import-Module .\Workflow.Cmdlets.dll`.

#### Invoke-Workflow

The `Invoke-Workflow` command executes a given workflow in-memory, within the PowerShell session.
In-memory execution follows the same rules as would executing the workflows in a distributed system, 
This is useful in cases where on-demand workflow execution is needed and allows attaching a debugger to the PowerShell process.

##### Parameters

|Name|Type|Required|Description|
|- |- |- |- |
|FullDirectoryPath|String|Yes|The folder containing the workflow assembly|
|Assembly|String|Yes|The assembly simple name|
|Namespace|String|Yes|The namespace containing the workflow class|
|Class|String|Yes|The workflow class name|
|Parameters|Hashtable|No|A hashtable containing a one-to-one mapping for workflow input properties|
|TimeoutSec|Int|Yes|The time after which this workflow will timeout|

##### Input
```powershell
Invoke-Workflow `
    -FullDirectoryPath "{WorkflowDirectory}"`
    -Assembly "TestWorkflows"`
    -Namespace "Grafixoft.Powertree.TestWorkflows"`
    -Class "EchoWorkflow"`
    -Parameters @{Input="echo"}`
    -TimeoutSec 6
```
##### Output
```powershell
EchoWorkflow received input: echo

   Status ReturnValue
   ------ -----------
Succeeded echo
```

#### Add-Workflow

The `Add-Workflow` command enqueues a new workflow instance for execution within the Operation Store.

Returns the added workflow's id.

##### Parameters

|Name|Type|Required|Description|
|- |- |- |- |
|ComponentName|String|Yes|The component containing the workflow assembly|
|ComponentVersion|String|No|If not set, the workflow runtime will resolve based on default and highest versions. Must be formatted as 0.0.0.0|
|Assembly|String|Yes|The workflow assembly simple name|
|Namespace|String|Yes|The namespace containing the workflow class|
|Class|String|Yes|The workflow class|
|Parameters|Hashtable|No|A hashtable containing a one-to-one mapping for workflow input properties|
|TimeoutSec|Int|Yes|The time after which this workflow will timeout|

##### Input
```powershell
Add-Workflow `
    -ComponentName "TestWorkflows"`
    -ComponentVersion "0.0.0.0"`
    -Assembly "TestWorkflows"`
    -Namespace "Grafixoft.Powertree.TestWorkflows"`
    -Class "EchoWorkflow"`
    -Parameters @{Input="echo"}`
    -TimeoutSec 6
```
##### Output
```powershell
Guid
----
0e85262d-be82-4e5e-a503-33b502b43724
```
#### Publish-Component
The `Publish-Component` command zips the specified directory and publishes it to the configured component store.
Make sure that your assemblies and *their dependencies* are included in the component directory.

##### Parameters

|Name|Type|Required|Description|
|- |- |- |- |
|ComponentName|String|Yes|The name of the component|
|ComponentVersion|String|Yes|The published version. Must be formatted as 0.0.0.0|
|Path|String|Yes|The component folder|
|IsDefault|Bool|No|If set, marks the newly published version as default|

##### Input
```powershell
Publish-Component `
    -ComponentName "TestWorkflows"`
    -ComponentVersion "1.0.0.0"`
    -Path "C:\TestWorkflows\bin\publish"`
    -IsDefault $true
```

#### Get-Workflow
The `Get-Workflow` command queries the Operation Store for details on a specific workflow.

##### Parameters

|Name|Type|Required|Description|
|- |- |- |- |
|Id|String|Yes|The workflow id|
|ShowAll|Switch|No|If set, returns raw workflow information|
|TreeView|Switch|No|If set, the output will be indented when showing tree structures|
|Depth|Int|No|If set, specifies that the workflow and its subworkflows up to the specified depth will be returned|

##### Input
```powershell
Get-Workflow -Id 1d85ba2c-1385-11eb-adc1-0242ac120002 -TreeView -Depth 1
```

##### Output
```powershell
Id           : f62decfd-12c7-4183-ab06-905dbcf5329b
ParentId     :
Status       : Not finished
ClassName    : BlenderRenderWorkflow
DateCreated  : 2020-10-21 11:11:39
DateFinished :
ReturnValue  :
Exception    :

    Id           : 626939f5-e718-4a2f-bdce-b3fc43e32031
    ParentId     : 7
    Status       : Not finished
    ClassName    : RenderFramesWorkflow
    DateCreated  : 2020-10-21 11:12:13
    DateFinished :
    ReturnValue  :
    Exception    :

    Id           : fc75ab1c-6815-4404-b3f9-9649b2ad4cd8
    ParentId     : 7
    Status       : Succeeded
    ClassName    : RenderFramesWorkflow
    DateCreated  : 2020-10-21 11:12:13
    DateFinished : 2020-10-21 11:33:12
    ReturnValue  :
    Exception    :

    ...
```

#### Stop-Workflow
The `Stop-Workflow` command sends a cancellation request for the specified workflow.

##### Parameters
|Name|Type|Required|Description|
|- |- |- |- |
|Id|String|Yes|The workflow id|

```powershell
Stop-Workflow -Id 1d85ba2c-1385-11eb-adc1-0242ac120002
```

