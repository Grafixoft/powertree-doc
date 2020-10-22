---
id: concepts
title: Concepts
---

### Concepts


### Workflows

Workflows are the only building block for large-scale distributed applications. A workflow is a primitive which uses code to define any piece of work. 
Each workflow can be viewed as a stateful actor which is executed across a pool of compute nodes and can have a number of subworkflows.

___
### Subworkflows

While workflows are extremely useful by themselves, a lot of use cases require work distribution and insight only known at runtime.
In PowerTree, this is achieved by the use of subworkflows. Subworkflows are defined in the same way as workflows, but they allow their parent to observe their state and take appropriate action. This two-way asyncronous communication between objects is at the core of the programming model and allows for a flexible decision making process, i.e. a parent workflow might decide to retry a failed subworkflow or send a notification when it has completed successfuly.

___
### Workflow Composition

[//]: # (TODO: Repetition)
At it's core, subworkflow orchestration is a form of the Divide-and-conquer algorithm. It allows for solving large, complex computational problems by breaking them down into smaller problems recursively until a solution tree is formed. Solving all nodes leads to solving the original large problem.

[//]: # (TODO: Better diagram)
![Workflow Composition](assets/workflow_composition.PNG)

___
### Activities

An activity or activation is the process of loading a workflow instance in the compute node memory and executing the logic contained within. A workflow can have many activations over the course of its lifetime and is most commonly triggered by workflow-specific logic, i.e. a new workflow has been submitted or some or all subworkflows have completed.

___
### Components

Components provide a way to package one or more workflow binaries and their dependencies.
They are versioned and allow easy rollbacks in case of bad revisions. All workflows are componentised 

___
### Component Deployment - TODO

Components can be distributed to compute nodes either before submitting the workflow request (eager deployment), or can be automatically downloaded at activation time (Just-In-Time deployment)
Component version is optional when submitting a workflow and each component can have a default version.
If no version is specified, it will be resolved to the default, or if a default is not set it will use the highest version