---
id: subworkflows
title: Subworkflows
---

### Working with Subworkflows

In the previous example we saw that the `ExampleWorkflow` called a `SubmitSubworkflows` method inside it's `DoWork` implementation. What `SubmitSubworkflows` actualy does is to enqueue a number of subworkflows through the [SubworkflowService](workflow-runtime#subworkflow-service). These subworkflows will be executed in parallel across available compute nodes after the parent has been deactivated.

```c#
private void SubmitSubworkflows(SubworkflowService subworkflows)
{
    foreach (char pieceOfWork in this.Parameter)
    {
        var childWorkflow = new ExpensiveComputationWorkflow
        {
            PieceOfWork = pieceOfWork
        };

        subworkflows.Enqueue(childWorkflow, timeout: TimeSpan.FromMinutes(10));
    }
}
```

The parent workflow also specifies a WaitAll `Continuation` meaning that it will become active again once all subworkflows have completed. 

It now knows that each subworkflow has either completed successfully, or failed and can take the appropriate action. 
The `SubworkflowService.GetUnobservedResults()` method returns a `WorkflowResult` collection which represents subworkflows completed in the time frame *between the parent's previous and current activation*.

In this case the parent collects all results from his subworkflows and simply sums them, or if any of them have failed, it fails itself.

```c#
private long ProcessFinishedSubworkflows(SubworkflowService subworkflows)
{
    IEnumerable<WorkflowResult> children = subworkflows.GetUnobservedResults();
    WorkflowResult[] failed = children
        .Where(c => c.Status != WorkflowStatus.Succeeded)
        .ToArray();

    if (failed.Any())
    {
        var errors = failed.Select(x => x.GetExceptionDetails()).ToArray();
        throw new WorkflowException("Subworkflows failed", errors);
    }

    long computationSum = children
        .Where(c => c.Status == WorkflowStatus.Succeeded)
        .Select(x => (long)x.GetReturnValue<int>())
        .Sum();

    return computationSum;
}
```

If any of them have failed, the workflow throws a `WorkflowException`.
It gives you quick error navigation through the ability to persist and retrieve cross-workflow, cross-node exceptions and execution stacks.

<!--DOCUSAURUS_CODE_TABS-->

<!-- Parent Workflow -->
```c#
[DataContract]
public class ExampleWorkflow : Workflow<long>
{
    [DataMember]
    private ExampleWorkflowState state;

    [DataMember]
    private long returnValue;

    [DataMember]
    public string PiecesOfWork { get; set; }
    
    [DataMember]
    public override long ReturnValue => this.returnValue;

    public override Continuation DoWork(WorkflowRuntime runtime)
    {
        switch (this.state)
        {
            case ExampleWorkflowState.FirstActivation:
                this.SubmitSubworkflows(runtime.Subworkflows);
                this.state = ExampleWorkflowState.ChildrenSubmitted;
                break;
            case ExampleWorkflowState.ChildrenSubmitted:
                this.returnValue = this.ProcessFinishedSubworkflows(runtime.Subworkflows);
                runtime.Logger.LogInformation(eventId: 1, "Distributed computation completed!");
                break;
        }

        return Continuation.WaitAll();
    }

    private void SubmitSubworkflows(SubworkflowService subworkflows)
    {
        foreach (char pieceOfWork in this.Parameter)
        {
            var childWorkflow = new ExpensiveComputationWorkflow
            {
                PieceOfWork = pieceOfWork
            };

            subworkflows.Enqueue(childWorkflow, timeout: TimeSpan.FromMinutes(10));
        }
    }

    private long ProcessFinishedSubworkflows(SubworkflowService subworkflows)
    {
        IEnumerable<WorkflowResult> children = subworkflows.GetUnobservedResults();
        WorkflowResult[] failed = children
            .Where(c => c.Status != WorkflowStatus.Succeeded)
            .ToArray();

        if (failed.Any())
        {
            var errors = failed.Select(x => x.GetExceptionDetails()).ToArray();
            throw new WorkflowException("Subworkflows failed", errors);
        }

        long computationSum = children
            .Where(c => c.Status == WorkflowStatus.Succeeded)
            .Select(x => (long)x.GetReturnValue<int>())
            .Sum();
        return computationSum;
    }
}
```

<!-- Subworkflow -->
```c#
[DataContract]
public class ExpensiveComputationWorkflow : Workflow<int>
{
    [DataMember]
    private int computedValue;

    [DataMember]
    public char PieceOfWork { get; set; }

    public override int ReturnValue => this.computedValue;

    public override Continuation DoWork(WorkflowRuntime runtime)
    {
        this.computedValue = this.DoExpensiveComputation(this.PieceOfWork);

        runtime.Logger.LogInformation(
            eventId: 9,
            "Single computation completed. Computed value for {input} is {value}",
            this.PieceOfWork.ToString(),
            this.computedValue.ToString());

        return Continuation.Default();
    }

    private int DoExpensiveComputation(char pieceOfWork)
    {
        // Well, it's not expensive, but it could be
        return pieceOfWork.GetHashCode();
    }
}
```

<!--END_DOCUSAURUS_CODE_TABS-->