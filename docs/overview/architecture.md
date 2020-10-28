---
id: architecture
title: Architecture
---

### Scale Unit

![Scale Unit](assets/scale_unit.PNG)

A PowerTree scale unit is a cluster of compute nodes and a single storage node. 
Compute nodes are highly available through redundancy and are responsible for executing workflows.
All of the nodes are peers without any master-slave architecture. This makes them highly scalable and fault-tolerant and you can add new machines without any interruptions to already running operations.

The storage node, also known as the operation store, serves as a work queue for the compute nodes and is a critical component for workflow persistence. It:

* Stores active workflows (hot data)
* Stores completed workflows (cold data)
* Serves as shared memory between compute nodes
* Has multiple implementations to target different scenarios [^1]

[^1]: Currently the only supported backends are in-memory (for testing scenarios) and SQL Server, though more will come in the near future. 
