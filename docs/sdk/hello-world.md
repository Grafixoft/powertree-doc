---
id: hello-world
title: Hello World!
---

### Your First Workflow

Once your development environment has been setup - you're ready to start developing.

As previously mentioned, workflows are extremely versatile and can encompass both the simplest and more complex use-cases. In this example we will explore the former - a Hello World workflow.

1. Create a `Demo.Workflows` class library project, targeting .NET Standard 2.0
2. Add a reference to `Workflow.Interface.dll`
3. Create a `HelloWorldWorkflow.cs` file and paste the following:

```c#
namespace Demo.Workflows
{
    using System;
    using Grafixoft.Powertree.Workflow.Interface;

    public class HelloWorldWorkflow : Workflow
    {
        public override Continuation DoWork(WorkflowRuntime runtime)
        {
            Console.WriteLine("Hello world!");

            return Continuation.Default();
        }
    }
}
```

*There are a few new concepts introduced here, but for now you only need to know 2 things - all workflows inherit from a base class defined in `Workflow.Interface` and all workflows are required to override the `DoWork` method, which encompasses this workflow's logic.*

4. Build the project and make note of its output folder containing the DLLs
5. Follow the steps outlined in the [Command-Line Interface](../how-to/cli#import) in order to start a PowerTree session in PowerShell.
6. From this point on, with a few additional steps the workflow can be hosted either on a compute node, or inside the PowerShell session. We'll explore the former since it's the simpler option.
7. Inside the PowerShell window replace the `{Build-Directory}` with your build output directory and run the following:

```powershell
 Invoke-Workflow `
    -FullDirectoryPath "{Build-Directory}"`
    -Assembly "Demo.Workflows"`
    -Namespace "Demo.Workflows"`
    -Class "HelloWorldWorkflow"`
    -TimeoutSec 5
```
8. You should see the following in the PowerShell window:

```
Hello world!

   Status
   ------
Succeeded
```