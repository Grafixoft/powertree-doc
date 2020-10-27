---
id: configuration
title: Configuration
---

### Execution Environments

An execution environment encapsulates all environment specific data under which workflows are executed.

In practice, this translates directly to the `ExecutionEnvironment.config` configuration file.
> `ExecutionEnvironment.config` is used by both workflow hosts and PowerShell cmdlets

```xml
<?xml version="1.0" encoding="utf-8" ?>
<configuration>
  <configSections>
    <section name="hostManager" type="Grafixoft.Powertree.Workflow.HostManager.Configuration.HostManagerSection, Workflow.HostManager"/>
  </configSections>

  <appSettings>
    <add key="ConnectionString" value="Data Source=(LocalDB)\MSSQLLocalDB; Integrated Security=SSPI; Initial Catalog=OperationStoreLocal"/>
    <add key="MaxActiveOperations" value="50"/>

    <add key="LocalComponentStoreDirectory" value="Z:\Storage\ComponentStore" />
    <add key="StoreTempDirectory" value="Z:\Storage\ComponentStoreTemp" />
    <add key="ComponentCacheDirectory" value="..\ComponentCache" />
    <add key="CacheTempDirectory" value="..\ComponentCacheTemp" />

    <add key="RuntimeLogBasePath" value=".\logs\runtime"/>
    <add key="ActivityLogBasePath" value=".\logs\activity"/>
    <add key="MinimumLogLevel" value="Information"/>
    <add key="LogFileSizeLimitBytes" value="1073741824"/>
  </appSettings>

  <hostManager>
    <processes>
        <add key="Host" fileName="Workflow.Host.exe" />
        <add key="Filebeat" fileName="filebeat\filebeat.exe" arguments="" />
    </processes>
  </hostManager>
</configuration>
```

|Name|Type|Required|Description|
|- |- |- |- |
|ConnectionString|String|Yes|The Operation Store connection string|
|MaxActiveOperations|Int|Yes|Specifies the maximum concurrency level for workflows|
|LocalComponentStoreDirectory|String|Yes|The path to the local component store base directory. Should point to a network share visible across all compute nodes|
|StoreTempDirectory|String|Yes|The path to the local component store temporary directory|
|ComponentCacheDirectory|String|Yes|The path to the local component cache base directory. Should point to a folder on the compute node|
|CacheTempDirectory|String|Yes|The path to the local component cache temporary directory|
|RuntimeLogBasePath|String|No|The base file path for runtime log files. Defaults to ./logs/runtime|
|ActivityLogBasePath|String|No|The base file path for activity log files. Defaults to ./logs/activity|
|MinimumLogLevel|[TraceEventType](https://docs.microsoft.com/en-us/dotnet/api/system.diagnostics.traceeventtype)|No|Defaults to Information|
|LogFileSizeLimitBytes|Int|No|The maximum size a log file reaches before rolling over to the next file. Defaults to 1073741824|