---
id: deployment
title: Deployment
---

All PowerTree binaries do not have any machine-specific installation and are designed to be runnable from the moment you unpack and apply the appropriate [configuration](configuration).

### Infrastructure Prerequisites

PowerTree can run on-premise or in the cloud, and is adaptive to a wide variety of deployment scenarios.

The least you would need to deploy a PowerTree cluster are one or more compute nodes, which can be any computer, virtual or otherwise, running inside the same network with access to a common, shared drive [^1]. You will also need a SQL Server[^2] 2016+ instance for the Operation Store database which should be accessible from within the nodes.

Logs from across the nodes are shipped to an existing Elasticsearch instance and can later be visualized in Kibana.

### Operation Store Deployment

The first step of setting up a PowerTree cluster is the SQL Server Operation Store deployment.

This is easily done through invoking a workflow in-memory:

1. Follow the steps outlined in [Command-Line Interface](../service/cli) in order to start a PowerTree session in PowerShell
2. Replace the parameters with your target values

```powershell
$DatabaseName = "OperationStoreLocal";

Invoke-Workflow `
    -Assembly "OperationStore.SqlServer"`
    -Namespace "Grafixoft.Powertree.OperationStore.SqlServer"`
    -Class "DeployOperationStoreSqlServerWorkflow"`
    -FullDirectoryPath (Join-Path $pwd ".\") `
    -Parameters @{`
        # Required
        ConnectionString="Data Source=(LocalDB)\MSSQLLocalDB; Integrated Security=SSPI; Initial Catalog=$DatabaseName";

        # Optional
        DatabaseInitialSize="100MB";`
        DatabaseMaxSize="1000MB";`
        DatabaseFilePath="$env:TEMP\$DatabaseName.mdf";`
        DatabaseFileGrowth="100MB";`
        LogInitialSize="100MB";`
        LogMaxSize="1000MB";`
        LogFilePath="$env:TEMP\$DatabaseName.ldf";`
        LogFileGrowth="100MB";`
    }`
    -TimeoutSec 60
```
3. Execute the command
4. You should see the following in the PowerShell window:

```
   Status
   ------
Succeeded
```

### Compute Node Deployment

The second step to deploying PowerTree is to deploy the workflow hosts themselves.

Bare metal deployment is the simplest, although most involved, deployment option. 
It consists of copying the PowerTree binaries and their respective configuration to all target compute nodes and setting up a daemon to keep the `Workflow.HostManager` process alive.

This, of course, is time-consuming and quickly becomes unmanageable once the system reaches a certain size.
For this reason, we've automated the process of deploying workflow hosts on an existing Service Fabric cluster and integrations with other service orchestrators are on their way.

#### Service Fabric Deployment

Workflow hosts can be easily deployed to an existing Service Fabric cluster, whether on-premise or in the cloud. This greatly simplifies the process of managing compute node instances and upgrading service versions.

Again, this can be accomplished by in-memory execution of a workflow.

The deployment workflow example below uses X.509 certificate authentication and assumes the appropriate Service Fabric certificate has been installed in the specified Certificate Store for the current user.

Depending on whether the application already exists, the deployment process will choose to perform a fresh install or an upgrade.
The upgrade process is monitored and executes health checks on every node. If a node's health is in an error or warning state the whole upgrade is rolled back.

All that's needed is to adjust the parameters as necessary and run the snippet below:

>Due to Service Fabric limitations the workflow can only be executed in **PowerShell 5**

```powershell
$paramsList = @{`
    # Connection to a primary node in the cluster
    ClusterConnection = "172.168.0.52:19000";`
    ImageStoreConnectionString = "fabric:ImageStore";`
    CompressPackage = $false;`
    # Temp path to store the package before uploading to Service Fabric
    PackageRootPath = "$env:TEMP\PowerTreePackage";`
    # Path to a folder where the runtime binaries are located
    SourcePath = "C:\PowerTree\";`
    # The certificate store name
    CertificateStoreName = "MY";`
    # The Service Fabric certificate thumbprint
    CertificateThumbprint = "ffffffffffffffffffffffffffffffffffffff";`
    # The common name of the Service Fabric certificate
    CertificateCommonName = "SFCertificate";`
    # If provided, it is copied in the package output folder
    ExecutionEnvironmentSourcePath = $null;`
    InstanceCount = -1;`
    IsSecuredCluster = $true;`
    ApplicationName = "PowerTreeApp";`
    Version = "1.0.0";`
    ServiceName = "WorkflowHost";`
    EntryPoint = "Workflow.HostManager.exe";`
}

Invoke-Workflow `
    -Assembly "Workflow.ServiceFabricDeployment"`
    -Namespace "Grafixoft.Powertree.Workflow.ServiceFabricDeployment"`
    -Class "RuntimeDeploymentWorkflow"`
    -FullDirectoryPath (Join-Path $pwd ".\Workflow.ServiceFabricDeployment\")`
    -Parameters  $paramsList `
    -TimeoutSec 600
```

Once the Operation Store and Workflow Hosts have been deployed, the PowerTree system is ready to use.

[^1]: Currently the only supported component storage is a shared drive, though extensibility is planned in the near future.
[^2]: Currently the only supported database is Microsoft SQL Server, though more will come in the near future.