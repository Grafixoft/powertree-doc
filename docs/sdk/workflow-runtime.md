---
id: workflow-runtime
title: Workflow Runtime
---

The `WorkflowRuntime` parameter of the `DoWork` method is responsible for providing essential runtime services to the workflow.

### Subworkflow Service

SubworkflowService class manages subworkflows for the current workflow  

```csharp
public abstract void Cancel(WorkflowId id);
```

Cancel issues a cancellation request for the specified subworkflow.   
If the subworkflow is active it will take some time before it is deactivated - the cancellation is not always instanteneous.  

```csharp
public abstract WorkflowId Enqueue(Workflow workflow, TimeSpan timeout);
```
Enqueue adds a subworkflow for asynchronous processing. Parent workflows cannot enqueue and observe a subworkflow result in the same activity.  

#### Observing Subworkflow State

```csharp
public abstract IEnumerable<WorkflowResult> GetUnobservedResults();
```

GetUnobservedResults provides the results for the subworkflows which have completed since the last activation of the caller workflow.  
Suitable for cases when the parent starts and handles many subworkflows uniformly without trying to distinguish them.  

```csharp
public abstract bool TryGetResult(WorkflowId id, out WorkflowResult result);
```

TryGetResult returns true and the result if the specified subworkflow has completed before activating the parent. Otherwise returns false and null.  
It is ok to call this method if the subworkflow has not completed yet.  

```csharp
public abstract WorkflowResult this[WorkflowId id] { get; }
```

Provides the result for a specific subworkflow. If the subworkflow has not completed yet this call will throw an exception.  
This is mostly suitable for cases when the parent has a single subworkflow so when the parent is activated it knows the subworkflow has completed.  

### Logger

The `WorkflowRuntime` provides a `Logger` instance to workflows. It allows developers and operators to trace workflow execution across nodes.
The activity logger is transparently enriched with properties relating to the current workflow, e.g. id of the workflow, the name of the executing class and which version and component does it belong to. These can be aggregated and searched down the line to make pinpointing problems easy.

It also supports Serilog's [Message Template Syntax](https://github.com/serilog/serilog/wiki/Writing-Log-Events#message-template-syntax)

```c#
runtime.Logger.LogInformation(eventId: 0, "Final result is {result}", result);
```

### Activity Cancellation

```c#
public virtual CancellationToken CancellationToken { get; }
```

The `CancellationToken` property of `WorkflowRuntime` is used to provide a cancellation notification to the workflow being executed.
Among others, the token will get signalled whenever the current workflow has been cancelled while active or when the host is shutting down.
While we do ask nicely, keep in mind that uncooperative workflows will be [//]:#(TODO) murdered and their bodies thrown into archive storage.

<!--DOCUSAURUS_CODE_TABS-->

<!-- Async Workflow -->
```c#
public override async Task<Continuation> DoWorkAsync(WorkflowRuntime runtime)
{
    await this.LongRunningOperationAsync(runtime.CancellationToken);

    return Continuation.Default();
}
```

<!-- Workflow -->
```c#
public override Continuation DoWork(WorkflowRuntime runtime)
{
    for (int i = 0; i < int.MaxValue; i++)
    {
        if (runtime.CancellationToken.IsCancellationRequested)
        {
            break;
        }

        this.result += this.SlowMethod();
    }

    return Continuation.Default();
}
```

<!--END_DOCUSAURUS_CODE_TABS-->

### Host Services

`HostServices` is a property on the workflow runtime interface of type `IServiceProvider`. It allows the workflow host (or a test) to provide object instances which the workflow can look up by type at runtime. Those objects are stable across multiple workflow activations and serializations - they are provided by the host and the runtime, not by de-serializing from the operation store. So, when it comes to instantiating external dependency the workflow can first consult the host services if such dependency already exists and only if it does not then it can go ahead and instantiate it. 

The pattern looks like this:
```c#
var dependency = runtime.HostServices.GetByType(type(IExternalService))
    ?? new ExternalServiceImplementation();
```
Usually the dependency is abstracted behind an interface which can be mocked or faked by the test code.