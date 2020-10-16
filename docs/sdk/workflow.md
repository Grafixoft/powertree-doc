---
id: workflow
title: Workflow
---

The `Workflow` classes defined in `Workflow.Interface` lie at the core of distributed computing with PowerTree. An executable workflow is just an implementation inheriting one of them.

Workflows have two variants - the `Workflow` and `Workflow<T>` classes with the difference being that the generic variant defines a `ReturnValue` property of type `T` allowing a user, or another workflow to access its result. They are also called action and function workflows respectively, referring to their .NET counterparts.

Both of them also have their `async` analogs, namely `WorkflowAsync` and `WorkflowAsync<T>`.

Note: *All workflows execute asynchronously and `WorkflowAsync` simply refers to the .NET task-based asynchronous programming model.*

```c#
[DataContract]
public abstract class Workflow
{
    public abstract Continuation DoWork(WorkflowRuntime runtime);
}

[DataContract]
public abstract class Workflow<T> : Workflow
{
    public abstract T ReturnValue { get; } 
}
```

### Workflow State

A workflow instance can keep internal state, which is then serialized and persisted for the next [activation](../overview/concepts#activities).
As seen in the definition above, `Workflow` classes are annotated with a `DataContract` attribute. 
Internally the PowerTree runtime uses Data Contract serialization and is subject to its rules and limitations. [See More](https://docs.microsoft.com/en-us/dotnet/framework/wcf/feature-details/data-contract-serializer)

Additionally, workflow state should be kept small since the shared node memory, i.e. the Operation Store is not intended to be used as blob storage and storing large amounts of data in a workflow can have performance implications. In cases where this is needed, it's recommended that a third-party or in-house solution be used instead of storing the data as workflow state.

#### Last Good Known State

An important concept in PowerTree, the last good known state refers to the mechanism through which workflow durability is achieved. If a compute node fails, e.g. due to a power outage, the last good known state allows the activation to be retried transparently on a different compute node as if the outage never happened.

### Workflow Parameters

Workflows can accept any number of parameters through public `[DataMember]` annotated properties.
Parameters can be passed to a workflow either by a user through the CLI, or through a parent workflow. More information on the latter can be found in the [next section](subworkflows)

### Workflow Continuation

The `DoWork` method returns a `Continuation` object, which specifies when the next [activation](../overview/concepts#activities) of the workflow will take place. 
* `Continuation.WaitAll()` specifies that the next activation should be made only after *all* subworkflows have completed
* `Continuation.WaitAny()` specifies that the next activation should be made as soon as *one or more* subworkflows have completed
* `Continuation.Default()` is the default continuation option. Equivalent to `WaitAny` continuation
* No `Continuation` - if a workflow has not submitted or doesn’t have pending subworkflows it is considered completed and `DoWork` is not called again on this instance. In this case the return value of `DoWork` is ignored
* More `Continuations` in the future

### Workflow Completion

All workflows are considered completed in the following scenarios:

1. If the `DoWork` method has exited without submitting subworkflows and there are no pending subworkflows the workflow has completed with `WorkflowStatus.Succeeded`
2. If the `DoWork` method threw an exception the workflow is completed with `WorkflowStatus.Failed`
3. If a workflow has been cancelled by a client or another workflow before completing its work it completes with `WorkflowStatus.Canceled`
4. If none of the above conditions have been met within the allotted timeout period it completes with `WorkflowStatus.Timeout`. All workflows have a timeout and it imposes an absolute time limit on workflows, allowing for recovery in unexpected situations, e.g. infinite loops
5. Child workflows are terminated prematurely if a parent workflow completes with a status other than `WorkflowStatus.Succeeded` before they manage to complete themselves. In this case they are assigned `WorkflowStatus.ParentTerminated`

The following example illustrates the topics discussed above.

### Example Workflow

```c#
[DataContract]
public class ExampleWorkflow : Workflow<long>
{
    // The workflow keeps internal state - a simple enumeration
    [DataMember]
    private ExampleWorkflowState state;

    // It stores a return value
    // Allowing a client or another workflow to access the result
    [DataMember]
    private long returnValue;

    public override long ReturnValue => this.returnValue;
    
    // It also accepts parameters
    // In this case a client has specified it when submitting
    [DataMember]
    public string Parameter { get; set; }

    // The workflow runtime calls DoWork multiple times over the lifetime of this instance
    public override Continuation DoWork(WorkflowRuntime runtime)
    {
        switch (this.state)
        {
            // First time DoWork is called
            // A good time to distribute work across the cluster through subworkflows
            case ExampleWorkflowState.FirstActivation:
                this.SubmitSubworkflows(runtime.Subworkflows);
                this.state = ExampleWorkflowState.ChildrenSubmitted;
                break;
            // Second time DoWork is called
            // The subworkflows have completed and the final result is calculated
            case ExampleWorkflowState.ChildrenSubmitted:
                this.returnValue = this.ProcessFinishedSubworkflows(runtime.Subworkflows);
                runtime.Logger.LogInformation(eventId: 1, "Distributed computation completed!");
                break;
        }
        
        // Wait for all subworkflows to complete before activating again
        // If there are no pending subworkflows, it completes successfully
        return Continuation.WaitAll();
    }
        
    // ...
}
```