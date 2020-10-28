---
id: concepts
title: Concepts
---

### Workflows

Workflows are the building block for large-scale distributed applications. A workflow is a primitive which uses code to define any piece of work. 
Each workflow can be viewed as a stateful actor which is executed across a pool of compute nodes and can have a number of subworkflows.

___
### Subworkflows

While workflows are extremely useful by themselves, a lot of use cases require work distribution and insight only known at runtime.
In PowerTree, this is achieved by the use of subworkflows. Subworkflows are defined in the same way as workflows, but they allow their parent to observe their state and take appropriate action. This two-way asynchronous communication between objects is at the core of the programming model and allows for a flexible decision making process, i.e. a parent workflow might decide to retry a failed subworkflow or send a notification when it has completed successfuly.

___
### Workflow Composition

Workflows are composed through subworkflows and at it's core is a form of the divide-and-conquer algorithm. It allows for solving large, complex computational problems by breaking them down into smaller problems recursively until a solution tree is formed. Solving all nodes leads to solving the original large problem.

![Workflow Composition](assets/workflow_composition_blue.PNG)
___
### Activities

An activity or activation is the process of loading a workflow instance in memory and executing the logic within. A workflow can have many activations over the course of its lifetime and is most commonly triggered by workflow-specific logic, i.e. a new workflow has been submitted or some or all subworkflows have completed.

___
### Components

Components provide a way to package one or more workflow binaries and their dependencies for use within PowerTree.
Quick recovery is important, so they are versioned and allow for rollbacks in case of bad revisions.

Component version is optional when submitting a workflow and each component can have a default version. If no version is specified, the workflow host will use the default, or if a default is not set - the highest available.

### Workflow Hosting

Workflows define what work needs to be done, but it's the responsibility of a workflow host to execute it.
Workflows can be hosted in many different ways - they can be in-process, in a PowerShell session or in the dedicated host process. 

After a client has submitted a workflow request within a scale unit, the first step of hosting the actual code is to deliver the workflow component to the compute node.
Components can be distributed to compute nodes either before submitting a request (eager deployment), or can be automatically downloaded at activation time (Just-In-Time deployment).

After the code has been loaded in memory, the runtime creates a workflow instance, executes its logic and can either complete it, or serialize and release it back to the Operation Store to wait for the next activation.
