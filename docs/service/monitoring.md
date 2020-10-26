---
id: monitoring
title: Monitoring & Tracing
---

When administering a live system it is crucial to ensure system operation is optimal and have the ability to react quickly and accurately to unforeseen circumstances.
A big part of this can be achieved through monitoring the application and system output logs.

Workflow logs contain all information needed to trace an error down to the specific workflow, its version, and the compute nodes it was executed on, allowing you to pinpoint the source and take appropriate action.

PowerTree comes with Elasticsearch integration and it allows for near real-time monitoring of system and workflow health.
Within seconds, you can visualize logs in Kibana and further search, filter, and analyze your logs for troubleshooting and open-ended exploration.

![Log Status Over Time](assets/log_over_time_component2.png)

That being said, it's easy to switch the log management solution. All logs are JSON formatted, allowing you to easily deploy your preferred log-shipping solution alongside the workflow host. 

```json
{
    "@t": "2020-10-21T10:13:16.0406763Z",
    "@m": "Starting directory search",
    "@i": "aab1db20",
    "@l": "Verbose",
    "HostName": "DESKTOP-OPU53IK",
    "WorkflowHostId": "Host49fcd69e-c5b8-4688-a8e7-ba309d0e3d31",
    "EventId": 0,
    "ThreadId": 9,
    "WorkflowId": "e85afd79-a5a7-4f45-a7cb-deb0613389b0",
    "ComponentName": "TestWorkflows",
    "ComponentVersion": "0.0.0.0",
    "AssemblyName": "TestWorkflows",
    "Namespace": "Grafixoft.Powertree.TestWorkflows",
    "ClassName": "SearchDirectoryWorkflow"
}
```